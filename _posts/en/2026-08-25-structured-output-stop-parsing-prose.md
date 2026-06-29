---
title: "Structured output: stop parsing prose, demand a schema"
date: 2026-08-25 00:00:00
lang: en
ref: structured-output-prompting
tags: [prompting, structured-output, reliability, workflow]
read_min: 4
excerpt_text: "You ask the model to extract fields and it hands you a friendly paragraph wrapped around a JSON blob. You write a regex to fish it out; it breaks on the next response. The fix isn't a nicer prompt — it's constraining the output to a schema and validating it at the boundary."
---

Back to that extraction endpoint — pull the line items out of an `orders` payload. I asked the model to "extract the line items as JSON." It did: *"Sure! I found 3 items:"* followed by a tidy blob inside a ```` ```json ```` fence. My `json.loads` choked on the prose, so I bolted on a regex to grab the fenced block. The next week a response came back with two fences and the regex grabbed the wrong one. I was writing parser archaeology against a thing that wasn't trying to be parsed.

The model wasn't misbehaving. I'd asked for an *answer* — a helpful, human-shaped reply — when what I needed was *data*.

## Ask for data, not an answer

"Extract the line items" is a request a helpful colleague answers in prose. To get data, constrain the *output format*, and there are two mechanisms for it:

- **Structured outputs** — you supply a JSON schema and the response is constrained to conform to it. No fences, no preamble, no "Sure!" — just the object.
- **Strict tool use** — the model fills in a typed tool call instead of writing free text. The arguments are validated against the tool's schema.

Either way the guarantee moves from *"the model usually complies"* to *"the response conforms to the shape, or you find out immediately."* That's the whole difference between a parser that works in the demo and one that survives production traffic.

## A schema is a contract, and it does double duty

The schema both *documents* the shape and *enforces* it. Spend the effort here — this is the real work, the same lesson as [job description over job title](/role-prompts-job-description/): the reliability lives in the spec, not in asking nicely.

```text
items: array of objects, each:
  sku:       string, required
  quantity:  integer, required        # not "three", not "3 units"
  category:  enum[food, electronics, other], required
  note:      string, optional
```

`enum` collapses an open-ended guess into a closed set you can branch on. `required` names what you cannot ship without. Typing `quantity` as an integer means you never again parse `"three"` downstream. The schema is where ambiguity goes to die.

## Validate at the boundary, always

Even with the output constrained, treat the response as untrusted input crossing into your system — the same posture as [reviewing AI-generated code](/reviewing-ai-generated-code-before-you-trust-it/). Parse it into your typed struct at the edge; reject on mismatch rather than letting a half-valid object flow three services deep into `billing`. And handle the empty case: a refusal or a truncated response means *no valid object*, not a default you silently invented. The schema makes the happy path clean; the boundary check is what keeps the unhappy path from becoming a 2 a.m. page.

## What it buys downstream

Parsing becomes deterministic — `json.loads` into a typed model, done, no regex that breaks on the second fence. Bad responses fail loudly at the edge instead of surfacing as a malformed row in `billing` a day later. And the schema is self-documenting: the next person reading the endpoint sees exactly what shape the model is held to.

## Impact

- **Parsing stops being fragile.** No regex archaeology, no string-fishing — the response either matches the typed struct or it's rejected at the door.
- **Errors surface at the boundary, not three services deep.** A malformed extraction is caught where it enters, not where it finally breaks something downstream.
- **The contract is visible.** The schema documents the exact shape, so the behavior is reproducible and the next reader doesn't have to reverse-engineer it from sample outputs.

## Decisions

- **Constrain the format; don't ask for it in prose.** A schema or a strict tool call beats "please reply with only JSON" — that instruction holds until the one response where it doesn't.
- **Put the effort into the schema.** `enum`, `required`, and correct types do more for reliability than any amount of prompt politeness.
- **Validate at the boundary anyway.** Constrained output lowers the odds of a bad shape; it doesn't remove your obligation to check before the data flows on.

## Limitations

- **A schema constrains shape, not truth.** The model can return a perfectly-valid object with the wrong `sku` in it — structure is not correctness, and you still need spot-checks on the values.
- **Refusals and truncation still happen.** A safety refusal or a hit token limit yields no valid object; without an explicit empty-case path you'll mistake "nothing" for "default."
- **Over-tight schemas backfire.** Forbid a field the data legitimately needs and the model is forced to drop or mangle it — model the real shape of your data, not the shape you wish it had.

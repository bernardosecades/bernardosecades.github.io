---
title: "Debugging with AI: hand it the evidence, not 'fix it'"
date: 2026-08-04 00:00:00
lang: en
ref: debugging-with-ai
tags: [debugging, prompting, distributed-systems, workflow]
read_min: 4
excerpt_text: "Asking AI to \"fix the bug\" gets you a confident patch for a bug you don't have. The model can't see your trace, so it pattern-matches to the most common cause and invents a root. Hand it the evidence and ask for ranked hypotheses instead."
---

An `orders` service that double-charges through `billing` — but only under load, maybe once in a few thousand requests. The kind of bug that doesn't reproduce on your laptop and survives three pairs of eyes. My first instinct was the lazy one: paste the function, *"there's a double-charge bug here, fix it."* What came back was a clean, confident patch — a mutex around a block that was never the problem.

The model didn't have the bug. It had a function and a label, so it did the only thing it could: it pattern-matched to the most common shape of a double-charge bug and patched that. Plausible. Wrong.

## Why "fix it" fails

With no evidence, the model can't *find* a root cause — it can only *invent* the most likely-sounding one. And it optimizes for a believable answer, not a true one. That's the trap: a confidently wrong patch is worse than "I don't know", because it reads like progress and you'll ship it. The less you give the model, the more it fills the gap with priors instead of your reality.

## Give it the evidence, not a verdict

Debugging is evidence work, and the model is only as good as what you put in front of it. Before asking for anything, hand over:

- **The actual signal** — the stack trace, the error, or the log lines *with timestamps* that show the anomaly. Not your paraphrase of them.
- **The real code path** — the two or three functions the request actually flows through, not only the one you suspect. The bug is often in the handoff, not the function.
- **What you've already ruled out** — "it's not client retries, the `gateway` log shows a single inbound call." This stops the model wasting a round on what you know is dead.
- **The right ask** — not "fix it" but "give me the most likely causes, ranked, and the cheapest way to confirm the top one."

## A prompt that works

```text
orders double-charges billing, ~1 in 3000 requests, only under load.
Here are the billing logs for one bad request (two POSTs, same
idempotency key, 40ms apart) and the two handlers in the path.
I've ruled out client retries — the gateway log shows a single inbound call.

Don't fix anything yet. Give me the 3 most likely root causes ranked
by likelihood, and for the top one the single log line or test that
would confirm or kill it.
```

"Ranked by likelihood" forces the model to reason about *your* evidence instead of reaching for a stock answer. "Don't fix anything yet" keeps it from skipping to a patch. And asking for the *cheapest confirmation* turns it from an oracle that guesses the answer into a partner that designs the next experiment.

## Let it design the experiment, then run it

The win isn't the patch — it's the loop. The model proposes a discriminating log line or a focused test; you run it — a flaky repro can churn [in the background](/claude-code-background-tasks/) while you keep working — and you paste the result back. Each round re-ranks against fresh evidence, not the model's prior, so two or three passes usually surface the real cause. (Here a role earns its keep as *calibration*, not credential: "trace which goroutine writes the row first" beats "you are a senior debugger" — same lesson as [job description over job title](/role-prompts-job-description/).)

## Impact

- **Fewer confident-wrong patches reach review.** The output is a ranked hypothesis with a confirmation step, not a fix nobody can vouch for.
- **The bug closes for a reason you can name.** You end with evidence that *this* was the cause, not a change that made the symptom disappear for now.
- **The transcript becomes a paper trail.** The ranked hypotheses and the experiments that killed them are the post-mortem, already written.

## Decisions

- **Withhold the verdict on purpose.** Ask for hypotheses, not fixes. The fix is the last step, not the first.
- **Give the path, not the suspect.** Hand over the whole request flow; the bug usually hides in the handoff between functions, which you can't see if you only paste the one you blame.
- **Make confirmation cheap before making it correct.** A log line that distinguishes two causes is worth more than a patch that assumes one of them.

## Limitations

- **Garbage evidence, garbage hypotheses.** If your logs don't capture the anomaly, the model is guessing alongside you — better instrumentation comes first.
- **Heisenbugs still bite.** Adding a log line can shift the timing enough to hide a race. The model can warn you about this, but it can't feel it happen.
- **It won't replace knowing your system.** The model ranks what's plausible; you still decide what's physically possible given how your services actually talk to each other.

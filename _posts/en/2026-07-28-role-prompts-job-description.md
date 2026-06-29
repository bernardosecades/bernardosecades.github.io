---
title: "Prompt roles: a job description beats a job title"
date: 2026-07-28 00:00:00
lang: en
ref: prompt-roles
tags: [prompting, claude, subagents, workflow]
read_min: 4
excerpt_text: "\"You are a senior expert\" barely moves a modern model — it's a credential sticker, not an instruction. A role earns its place when it encodes behavior, constraints and audience: what to do, what not to do, who you're talking to."
---

For a while I opened prompts with the responsible-sounding incantation: *"You are a senior engineer with 20 years of experience. Refactor this function."* It felt like I was setting the model up to do its best work. The output came back clean, confident — and no better than when I dropped the first sentence entirely.

That sentence wasn't an instruction. It was a credential sticker. And the model doesn't get more capable because I told it it was.

## Why the title does little

There's a real memory behind the habit. On older, smaller models, a persona prefix moved the needle — it nudged a weak model toward a register it otherwise wandered away from. So "you are an expert X" became muscle memory, and the muscle outlived the reason.

On a modern Claude (3.5 onward, and the 4.x family) the effect on *objective accuracy* is marginal to nil. A 2024 study on personas in system prompts put numbers on it: adding a persona didn't reliably improve measurable performance, and on some tasks it slightly hurt. The intuition is simple — the model already knows how to answer a coding question well. A job title in front of the task doesn't add a skill it was missing. It just spends tokens.

## What a role actually buys you

A role earns its place when it stops being a label and starts encoding a *job description* — three things, specifically:

- **Behavior** — what to do, and what *not* to do.
- **Constraints and stance** — the posture it judges from ("you have no stake in this").
- **Audience** — who the answer is for, which sets the level of detail and the register.

The blog already has a role that works, and it works for exactly these reasons. The [writer/reviewer pattern](/claude-code-writer-reviewer/) leans on a reviewer agent whose prompt opens:

```text
You are a code reviewer. You did not write this code and you have
no stake in it. Your job is to find problems, not to approve.
```

The word "reviewer" isn't what does the work. The work is in the rest: a stance (*no stake*), a behavior (*find problems*), and an explicit anti-goal (*not to approve*). Strip that to just "You are a senior reviewer" and you're back to a sticker.

## Bad vs good

The cargo-culted version:

```text
You are a world-class senior Go engineer. Refactor this function.
```

The version that actually constrains the work:

```text
Refactor this function. Keep the public signature. No new deps.
Optimize for readability over cleverness. Explain any behavior
change in one line.
```

The second prompt has no role at all and lands better — because the gains live in instructions, constraints, and output format, not in a self-image. The role only re-enters when it's pulling its weight on *tone or audience*:

```text
Explain this stack trace to a backend dev with no Go experience —
assume they know HTTP but not goroutines.
```

That last one isn't a credential. It's a calibration: it tells the model who's listening, and the explanation reshapes itself accordingly.

## When a role helps (and when it's noise)

| Use a role for… | Skip it for… |
|---|---|
| Tone & voice (skeptical reviewer, plain-language teacher) | Raw accuracy — "you are an expert" ≠ correctness |
| Audience calibration ("explain to someone who…") | Factual / coding tasks where clear instructions suffice |
| Behavioral framing in reusable agents & subagents | One-off prompts where the instruction already says it all |

## Impact

- **Prompts get shorter and more honest.** The effort moves to the part that matters — the instruction — instead of a decorative preamble that reads like work but isn't.
- **Measurable tasks improve for the right reason.** Accuracy climbs from context, examples, and an explicit output format. When it climbs, you know *why*, and you can reproduce it.
- **The roles you do keep are the ones worth keeping.** In an agent or subagent definition, a behavioral role gets versioned and reused — like the `diff-reviewer` — instead of being re-improvised in every chat.

## Decisions

- **Separate the credential role from the behavioral role.** "You are a senior expert" is the first; "you have no stake, find problems, don't approve" is the second. That line is the whole difference between cargo cult and technique.
- **Spend the elaborate roles where they're reused.** A committed agent prompt earns a careful persona. A throwaway one-off doesn't — just write the instruction.
- **To raise accuracy, invest in instructions, context, examples, and output format — not in a persona.** That's where the gains actually are.

## Limitations

- **This isn't "roles don't work."** They work for tone, audience, and framing. What deflates is the role as an *accuracy* booster.
- **The exact effect is model- and task-dependent.** On a small or local model, a persona can still earn more of its keep than it does on a frontier Claude.
- **A good role won't rescue a bad instruction.** If the task itself is underspecified, no amount of persona fixes it — the role is seasoning, not the meal.

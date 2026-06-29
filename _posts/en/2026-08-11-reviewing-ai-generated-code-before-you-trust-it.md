---
title: "Reviewing AI-generated code: what I check before I trust it"
date: 2026-08-11 00:00:00
lang: en
ref: reviewing-ai-generated-code
tags: [code-review, failure-cases, distributed-systems, workflow]
read_min: 4
excerpt_text: "AI code fails differently than human code: it's plausibly wrong, not obviously wrong. It reads clean, the tests are green, and the bug is in the seam it couldn't see. Here's the checklist I run before I trust a diff I didn't type."
---

A diff lands from my AI pair: a new handler in `orders`, clean naming, tests green, reads like something I'd have written on a good day. The instinct is to skim it and approve. That instinct is the whole problem — *looks done* is exactly the failure mode, because the model optimized for code that **looks** correct, and it got very good at it.

A junior writes obviously-wrong code: a typo, a missing return, something the compiler or a glance catches. AI writes *plausibly*-wrong code: the right shape, the wrong seam. So I stopped reviewing it like human code and started reviewing it for the specific ways it goes wrong.

## Why AI code needs a different pass

The model wrote the diff with a narrow view: the function I pasted, maybe the file. It can't see the caller three services away, the migration that has to ship with it, or the race it just introduced. It fills those gaps with what's *plausible*, not what's *true in your system* — the same trap as asking it to ["fix the bug"](/debugging-with-ai-hand-it-the-evidence/) blind. Review is where you supply the system-level view it never had.

## What I check first

These are the AI-specific tells, in the order they bite:

- **The error paths, not the happy path.** The model loves the path where everything works. Does it handle the timeout, the `nil`, the half-written row — or just `return err` and move on? Swallowed errors are the most common thing I send back.
- **The blast radius beyond the diff.** It changed the function; did it update the callers, the migration, the `billing` consumer that reads the output? The bug is usually in what the diff *didn't* touch.
- **Concurrency and shared state.** A goroutine added without a thought for the race; a lock that guards the write but not the read. In a distributed path this is where "works on my laptop" lives.
- **Secrets and injection.** A token in a log line, SQL built by string concat, a scope widened "to make it work." The model will cheerfully do all three.
- **Silent scope creep.** Did it reformat an unrelated block, change a default, or "improve" something I never asked about? Unrequested changes are where regressions hide.
- **Tests that assert the mock, not the behavior.** Green doesn't mean tested. If the test only proves the mock was called, it proves nothing.

## Make the model review its own seams

The first pass doesn't have to be human. Before I read the diff, I ask the model adversarially — not "is this good?" but *"list what breaks in production that this diff doesn't handle, ordered by likelihood."* That's the [writer/reviewer pattern](/claude-code-writer-reviewer/): a reviewer prompt with no stake, reading the diff cold, hunting for problems rather than approving. It catches the mechanical seams cheaply. What stays on me is the system-level call — whether *this* change is right for how `orders` and `billing` actually behave under load.

## Impact

- **Fewer plausible-but-wrong diffs reach `main`.** The review targets where AI code actually fails, so the catch rate goes up without the review taking longer.
- **The review gets faster, not slower.** A fixed checklist of tells beats re-reading every line as if it were equally suspect.
- **Trust becomes calibrated.** I stop oscillating between rubber-stamping and rewriting everything, and review the seams the model couldn't see.

## Decisions

- **Review AI code for its failure modes, not as if a human wrote it.** The tells are different, so the checklist is different.
- **Let an adversarial pass run first, keep the system call for the human.** The model finds mechanical problems; you decide whether the change fits the system.
- **Treat "looks done" as a red flag, not a green light.** The cleaner it reads, the more deliberately I check the seams.

## Limitations

- **A checklist isn't a substitute for understanding the change.** If you don't know what the code is supposed to do, no list catches a subtly wrong design.
- **The adversarial pass shares the model's blind spots.** It reads the diff cold but still can't see your whole system — it won't know the `billing` consumer exists unless you tell it.
- **It scales with diff size.** A 40-line diff gets a real review; a 2,000-line one gets skimmed no matter the checklist. Small diffs are still the actual fix.

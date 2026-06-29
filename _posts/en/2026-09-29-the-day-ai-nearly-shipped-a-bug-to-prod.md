---
title: "The day AI nearly shipped a bug to prod — and what I changed"
date: 2026-09-29 00:00:00
lang: en
ref: ai-nearly-shipped-a-bug
tags: [failure-cases, code-review, reliability, workflow]
read_min: 5
excerpt_text: "A clean AI diff, a green test suite, a confident explanation — three green lights, and I almost approved a double-charge bug. The lesson wasn't 'don't use AI.' It was that the code, the tests, and the explanation all came from one model, so they never corroborated each other."
---

Friday afternoon, moving fast. The model had produced a tidy change to the idempotency handling on the `orders` → `billing` charge path — the kind of fix I'd asked for a hundred times. The diff read well. The test suite was green. When I asked it to explain the change, the explanation was crisp and convincing. I had my cursor on approve. The only reason I didn't is a vague, unearned nag that I'd seen this shape of "everything's fine" before — and that nag, not my process, is the only thing that caught it.

The change was wrong. Not obviously wrong — *plausibly* wrong, which is worse.

## What actually happened

In tightening the idempotency check, the model had quietly dropped a guard that only mattered under concurrent retries. In the single-request world the tests exercised, it was flawless. Under two near-simultaneous retries of the same charge — the exact scenario the guard existed for — it would double-charge. Every signal in front of me said *ship it*: the diff was clean, the suite was green, the author was confident. The bug lived precisely in the gap none of those signals covered.

## The real lesson: AI failures are correlated

Here's what took me a while to see. I'd treated three things as independent confirmation — the code looks right, the tests pass, the explanation makes sense — when all three came from the *same model*. The diff that fooled my review also wrote the tests that went green (they asserted the single-request behavior, never the concurrent one) and also produced the explanation that talked me past my doubt. That's not three witnesses agreeing. It's one source agreeing with itself, three times, in three formats.

This is the trap underneath every individual failure mode I'd written about: [plausibly-wrong code](/reviewing-ai-generated-code-before-you-trust-it/), [green tests that prove nothing](/testing-with-ai-green-doesnt-mean-tested/), a confident voice. Each is survivable alone. Stacked, they manufacture a consensus that feels like safety and isn't — because the consensus has a single point of origin.

## What I changed

The fix isn't more vigilance in the moment. It's making sure at least one check comes from *outside the model's own loop*:

- **I read for behavior, not for polish.** Not "is this diff clean" but "what does this do when `billing` gets two retries 40ms apart." Cleanliness is the model's strength and exactly the wrong thing to be reassured by.
- **The decisive test is one I own.** Written against the spec — *the guard must hold under concurrent retries* — not generated against the code it's meant to check. A spec-first test can't be talked out of failing.
- **"Looks done" is the brake, not the green light.** The smoother everything feels, the more deliberately I now check the seam — because smooth is what correlated confidence feels like from the inside.

And when I want a second opinion, it comes from a [reviewer with no stake](/claude-code-writer-reviewer/) reading the diff cold, or from a human — something that didn't write the code, so its agreement actually counts as a second source.

## What I didn't change

I still use AI for all of it, every day — the code, the tests, the first draft of the explanation. The leverage is real and I'm not giving it back. The change was one habit, not a retreat: stop letting the model grade its own work, and supply exactly one check it can't author. That's cheap insurance, not a loss of speed.

## Impact

- **One near-miss didn't become an incident.** The double-charge was caught at review instead of in production — and the catch is now a process, not a lucky nag.
- **The independent check is cheap and load-bearing.** One spec-first test plus a behavior-focused read costs minutes and breaks the correlated-confidence trap that no amount of staring at a clean diff would have.
- **My trust got calibrated, not withdrawn.** I lean on AI exactly as hard as before, but I stopped counting its self-corroboration as evidence.

## Decisions

- **Treat AI's code, tests, and explanation as one source.** They don't independently confirm each other; require at least one check from outside that loop.
- **Own the test that matters.** Generated tests are fine for coverage; the one guarding the behavior you're nervous about, you write against the spec.
- **Slow down where it feels done.** The strongest "looks fine" signal is exactly where correlated failure hides — make smoothness trigger scrutiny, not relief.

## Limitations

- **A nag is not a process — and I almost didn't have one.** The honest version of this story is that luck caught it; the changes are what convert "got lucky once" into "would catch it next time," but they only work if I actually run them under deadline pressure.
- **Outside-the-loop checks have a cost.** A spec-first test and a careful behavioral read take time; on genuinely low-stakes changes, that ceremony isn't always worth it — judgment still required.
- **This generalizes only so far.** Correlated failure is the lens for *AI-authored* work; it's not a universal theory of bugs, and over-applying it would make every clean diff feel like a trap when most of them aren't.

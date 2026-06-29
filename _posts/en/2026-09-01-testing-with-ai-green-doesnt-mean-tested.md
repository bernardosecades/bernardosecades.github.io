---
title: "Testing with AI: green doesn't mean tested"
date: 2026-09-01 00:00:00
lang: en
ref: testing-with-ai
tags: [testing, failure-cases, reliability, workflow]
read_min: 5
excerpt_text: "Ask AI for tests and you get a full green suite that feels like coverage. Then you read them: most assert that a mock was called, one re-encodes the bug you have as the expected value, and none hit the case that actually breaks. Green, and worthless. Here's how to get tests that can fail."
---

I asked the model to write tests for the charge path between `orders` and `billing`. It produced fourteen, all green, and they looked thorough — setup, teardown, descriptive names. Then I read them. Most asserted that a mock had been called with certain arguments. One pinned the off-by-one I actually had as the *expected* value. None of them touched the concurrent double-charge that was the whole reason I was nervous about that code. Fourteen green tests that proved nothing.

A green suite feels like safety. AI-written tests are very good at *feeling* like safety while testing almost nothing — and the failure is specific enough to recognize once you've seen it.

## Why AI tests go green and prove nothing

The model optimizes for tests that pass and look complete, and the cheapest way to make a test pass is to assert what the code *does*, not what it *should*. Three shapes recur:

- **Tautological.** The test asserts that a mock was called, or mirrors the implementation step for step. It passes because it's a reflection of the code, not a check on it — rewrite the function wrong and the test still goes green.
- **Happy-path only.** It exercises the case the code already handles and skips the timeout, the empty input, the duplicate — exactly where the bug lives.
- **Bug-confirming.** It snapshots the *current* behavior as the expected value. If the current behavior is wrong, the test now guards the bug and fails the day you fix it.

## Write the test first, then the code

The single change that kills most of these is order: get the failing test *before* the implementation exists. A test written against the implementation can mirror it; a test written against the spec has nothing to mirror, so it can't be tautological. Have the model propose the test from the requirement, confirm it **fails for the right reason** — not because of a typo, because the behavior genuinely isn't there yet — and only then let it implement. Red, then green, means the green actually meant something.

## Aim it at the edges you can't see

This is the real upside, and it's the flip side of the failure. The model is *good* at enumerating boundary cases a tired human skips — empty input, the timeout, the duplicate request, the off-by-one at a range edge, the concurrent path. The trick is to ask for them explicitly instead of letting it write confirmation tests. Same move as [handing it the evidence when debugging](/debugging-with-ai-hand-it-the-evidence/): don't say "write tests," say *"list the cases that would break this that the happy path doesn't cover, ranked by likelihood."* Then write tests for those. The model surfaces the duplicate-charge case I'd have skipped at 6 p.m.; I decide which ones are real.

## Read every test like it's lying

The discipline from [reviewing AI-generated code](/reviewing-ai-generated-code-before-you-trust-it/) applies to the tests too — maybe more, because a bad test is worse than no test: it's false confidence with a green checkmark. For each one, ask two questions: does this assertion check *behavior* or *the mock*? And **would it fail if the code were wrong?** Mutate the implementation in your head — flip a comparison, drop a guard — and if the test would still pass, it isn't testing anything. A test that can't fail is documentation at best.

## Impact

- **Green starts meaning something again.** Tests written spec-first and read for can-it-fail catch real regressions, instead of decorating the CI run with confidence nobody earned.
- **Coverage lands where bugs are.** Asking explicitly for the breaking cases pushes tests toward the timeout and the duplicate, not a fifth variation of the happy path.
- **The edge cases I'd have skipped get caught.** The model's enumeration is genuinely better than tired-me at 6 p.m. — used deliberately, that's the win.

## Decisions

- **Test-first whenever the behavior is specifiable.** A red-then-green cycle structurally prevents the tautological and bug-confirming shapes; writing tests after the code invites both.
- **Ask for the breaking cases, not "tests."** "Write tests" gets confirmation; "what breaks that the happy path misses" gets coverage.
- **Read every assertion for can-it-fail.** A test that passes regardless of whether the code is correct is removed or rewritten — it's worse than absent.

## Limitations

- **Test-first doesn't fit everything.** Exploratory code, UI tweaks, and throwaway scripts don't have a crisp spec to write a failing test against — forcing TDD there is ceremony.
- **The model proposes; it doesn't know your domain.** It can list plausible edge cases, but which ones are *real* for your system — which states are actually reachable — is still your call.
- **Coverage is not correctness.** A green suite that can fail is necessary, not sufficient; the cases you never thought to ask about stay untested no matter how the tests were written.

---
title: "Token economics: measure before you optimize"
date: 2026-09-22 00:00:00
lang: en
ref: token-economics
tags: [cost, tokens, performance, workflow]
read_min: 5
excerpt_text: "The bill creeps up and the instinct is to make the prompt shorter. But shaving words is optimizing the wrong line — the real spend is structural: context resent every turn, no caching, the wrong tier. Measure first, and the expensive line is almost never the one you'd guess."
---

The token bill on the `orders` extraction pipeline crept up, and my first instinct was the obvious one: trim the prompt — tighten the wording, cut the examples, make it lean. Before I did, I pulled the actual usage breakdown. The prompt prose was about 2% of the spend. Eighty percent was re-sending the same large context on every turn, uncached. An afternoon spent word-golfing the prompt would have been an afternoon spent on the wrong line entirely.

Token cost is an engineering problem with the same trap as every other performance problem: the bottleneck is almost never where your intuition points, and optimizing before you measure is just guessing with extra steps.

## Measure before you cut

You can't optimize a bill you haven't broken down. Get the *exact* counts — the API reports tokens per call, so there's no excuse for eyeballing or reaching for a generic tokenizer that's wrong for the model. Then look at the breakdown along the axes that matter: input vs output, cached vs uncached, per-call vs per-loop. The expensive line reveals itself immediately, and it's usually structural — a re-sent context, an uncached prefix, a model tier too high for the task — not the wording you were about to agonize over.

## The big levers, in order

Once you've measured, spend effort top-down:

- **Caching.** A stable prefix sent on every request — system prompt, instructions, the big shared document — should be cached; a cache read costs a fraction of fresh input. And watch for *silent* misses: a timestamp or a request ID near the front of the prefix invalidates the cache every call and quietly doubles your bill while looking fine.
- **The context you resend.** Long agentic loops resend the whole history every turn, so a bloated transcript is a tax you pay on every step. Prune or compact stale tool output before it compounds — this is the [context window as a budget](/claude-code-context-window/), seen from the cost side.
- **The tier.** A narrow, high-volume task running on the flagship is the single most common overspend — [match the tier to the task](/picking-the-right-model-for-the-task/).
- **Fan-out.** N agents is N× the tokens; [only fan out when it pays for itself](/multi-agent-orchestration-when-its-worth-it/).

The micro-savings — like dropping shell output straight in with the [`!` shortcut](/claude-code-bang-shortcut/) instead of round-tripping through the model — are real, but they're step five, not step one. Don't start there.

## When the savings aren't worth it

Here's the part that's easy to forget mid-optimization: tokens are cheap, and your time isn't. Shaving 10% off a call that costs cents and runs twice a day is a net loss the moment it costs you an afternoon. Optimize the calls that are **hot** (high volume) or **fat** (huge context); for everything else, the cheapest move is to leave it alone. Premature token-golfing is the same mistake as premature performance tuning — effort poured into a line that was never the problem, paid for in the time you didn't spend on one that was.

## A cheap habit that pays

Log usage on every call — input, output, cached, uncached — so the breakdown is always sitting there when the bill moves. It costs almost nothing to add and it changes the failure mode: "the bill went up" stops being an investigation you run under pressure and becomes a number you already have. The afternoon I *didn't* spend word-golfing the prompt, I got back because the usage log told me where the 80% actually was.

## Impact

- **Optimization lands on the expensive line.** Measuring first redirects the effort from prompt wording (2%) to resent context (80%) — same hour of work, an order of magnitude more saved.
- **Silent cache misses get caught.** Watching cached-vs-uncached turns a quietly doubled bill into a one-line fix instead of a slow, unexplained creep.
- **Cheap calls get left alone.** Knowing which calls are hot or fat means the rest don't soak up engineering time that costs more than the tokens would.

## Decisions

- **Break down the bill before changing anything.** The exact per-call counts decide where to spend effort; intuition reliably points at the wrong line.
- **Work the levers top-down.** Caching and resent context before tier before fan-out before micro-savings — the order is roughly the order of impact.
- **Stop when the engineering time outweighs the bill.** A cheap, rare call is not worth an afternoon; optimize hot or fat, leave the rest.

## Limitations

- **Caching has its own rules.** A cached prefix only helps if it's genuinely stable and long enough to qualify; misuse it and you pay write premiums for reads that never come.
- **Cheaper tokens can cost accuracy.** Trimming context or dropping to a smaller tier saves money right up until it changes the answer — measure the quality, not just the bill.
- **Prices and ratios move.** The *order* of the levers is durable; the exact break-even between "optimize" and "leave it" shifts as pricing and your call volume change — re-measure, don't memorize.

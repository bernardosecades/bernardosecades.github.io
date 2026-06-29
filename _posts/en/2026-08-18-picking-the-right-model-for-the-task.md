---
title: "Picking a model: stop defaulting to the biggest one"
date: 2026-08-18 00:00:00
lang: en
ref: choosing-the-right-model
tags: [models, cost, prompting, workflow]
read_min: 4
excerpt_text: "The instinct is that the biggest model is the safe choice. But for classification, extraction, and routing, the cheap tier is usually indistinguishable on accuracy and far faster — and when accuracy is short, the fix is often a better prompt, not a bigger model."
---

An extraction endpoint — pull the line items out of an `orders` payload into structured JSON — running on the flagship model. It works great. It also costs roughly five times what it needs to and its p95 latency is double what it could be. The model was never the hard part of that task; the schema and the prompt were. I'd reached for the biggest model out of instinct, the way you'd reach for the strongest tool in the box without checking whether the bolt actually needs it.

Picking a model is a real engineering decision with a cost and latency bill attached, not a default.

## The tiers, and what each is for

The lineup sorts into three tiers, and the price gap between them is large — the cheapest tier runs at roughly a fifth the per-token cost of the top one, with the mid tier in between:

- **Top tier (Opus-class, and the frontier reasoning model above it).** The hardest reasoning, long-horizon agentic work, judgment calls where being wrong is expensive. You pay the most per token and wait the longest.
- **Mid tier (Sonnet-class).** The workhorse: high-volume production, solid reasoning, much faster and cheaper. Where most application traffic should live.
- **Cheap tier (Haiku-class).** Fast and inexpensive — classification, extraction, routing, short summaries, anything scoped and well-specified.

## Match the tier to the task, not your anxiety

The pull toward the biggest model is anxiety, not analysis: *bigger feels safer*. But on a well-specified, narrow task — label this ticket, pull these fields, route this request — the cheap tier is usually indistinguishable from the flagship on accuracy, while being several times faster and cheaper. The extra capability you paid for had nothing to do on that task.

Spend the top tier where the task genuinely needs it: multi-step reasoning, an open-ended agentic loop, code review, a judgment call with real downside. That's where the gap between tiers actually shows up in the output.

## Effort is a second dial

Model choice isn't the only lever. On the frontier models, an **effort** setting (low / medium / high) trades thoroughness for tokens and latency *independently* of which model you picked. So "top tier at low effort" and "mid tier at high effort" are both real, distinct options — sometimes the cheaper, faster one wins on the same task. Sweep it on your own workload rather than leaving it at the default.

## How I actually pick

Start one tier *below* your instinct. Measure on a real eval set — your prompts, your data, your definition of correct — not on vibes. Move up only when the numbers say accuracy demands it. And when accuracy is short, check the prompt first: clearer instructions, a sharper schema, a couple of examples ([job description over job title](/role-prompts-job-description/)) close the gap more often than a bigger model does — and they don't add cost on every call forever.

## Impact

- **The bill tracks the work.** High-volume narrow tasks run on the cheap tier; the top tier is reserved for what needs it, instead of paying flagship rates for a classifier.
- **Latency drops where users feel it.** A cheaper tier on a scoped task is often several times faster — real p95 improvement for free.
- **Model choice becomes a measured decision.** Backed by an eval set, you can say *why* a given route runs on a given tier, and revisit it when the task changes.

## Decisions

- **Default down, not up.** Start a tier below instinct and let the eval set pull you up, rather than starting at the top and never questioning it.
- **Treat effort as a separate axis from the model.** Tune both; the cheaper-and-faster combination sometimes wins outright.
- **Fix the prompt before upgrading the model.** A bigger model is the expensive answer to a problem a better instruction often solves for free.

## Limitations

- **You need a real eval set to do this honestly.** Without one, "the cheap tier is fine" is a guess — and the wrong guess ships subtle errors at scale.
- **The cheap tier has a ceiling.** Push a genuinely hard reasoning or long-horizon task onto it and you'll get confident, wrong output — the savings evaporate the moment correctness matters.
- **Tiers and prices move.** The *shape* of the decision is durable; the exact ratios and which model sits in each tier are not — re-baseline when the lineup changes.

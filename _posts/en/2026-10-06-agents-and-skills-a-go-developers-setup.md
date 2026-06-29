---
title: "Stop building a \"Go agent\": organize AI tooling by task shape, not language"
date: 2026-10-06 00:00:00
lang: en
ref: agents-skills-go-dev
tags: [agents, skills, workflow, go]
read_min: 5
excerpt_text: "The instinct when you adopt agents and skills as a Go developer is to build Go versions of everything — a Go reviewer, a Go test bot. You end up with a pile of overlapping tools and never remember which to invoke. The axis was never the language."
---

The instinct when you adopt agents and skills as a Go developer is to build *Go* versions of everything: a Go reviewer, a Go test generator, a Go observability checker. Six months later you have a drawer full of overlapping tools and you never remember which one to reach for. The mistake isn't the tools — it's the axis you organized them on. The language was never the right axis.

The right axis is the **shape of the task**. Agents and skills are two primitives with completely different physics, and once you sort your work by which physics it needs, the "do I make a Go thing?" question dissolves.

## Two primitives, different physics

A **skill** loads into your main context automatically when your request matches its `description`. It rides along *while you type* — it's knowledge, a convention, a checklist, a procedure. It shapes the code as it's being written. There's no extra step and no waiting for a verdict.

An **agent** runs in an isolated context, in parallel, and hands back only a conclusion. It's an investigator. It never sees what you're doing in your own window, and you never see its forty intermediate file reads — just the dictamen at the end.

That difference is the whole decision:

> Do I need it *while I'm writing*? → **skill.** Is it a *verdict I want afterward* that can run blind and in parallel? → **agent.**

Notice the language never entered into it.

## My daily Go loop, mapped

Walk a normal ticket and the two primitives fall into different moments.

**Orienting** (start of the ticket). I send an [explore-style agent](/claude-code-subagents/) to fan out across the `orders` and `billing` repos and find where a thing actually lives. This is an agent precisely because I *don't* want forty file dumps polluting my context — I want one answer.

**Writing** (the bulk of the work). This is skill territory, because skills are inline:

- A **Go idioms** skill — error wrapping, `context.Context` propagation, table-driven tests. Applied as the code comes out, not flagged later.
- An **observability** skill — how `orders` instruments with OpenTelemetry: span naming, propagating context through the call, the standard attributes. I want this at the moment I write the handler, not as a verdict that arrives after I've already gotten it wrong.
- A **dependencies** skill — the `go.mod replace` + `go mod vendor` dance when I point at a local `auditlib` checkout. Procedural and easy to get subtly wrong: a perfect skill.

**Verifying** (before the PR). Now agents earn their keep. I fan out a reviewer, a security auditor, and a test reviewer over `git diff main...HEAD` — [in parallel, each returning its slice](/multi-agent-orchestration-when-its-worth-it/), and I synthesize. This work is isolated and parallel by nature: none of those reviewers needs to be in my head while I code, and none needs to see the others.

**Operating** (when something breaks). Skills again — the procedural knowledge that wraps a CLI to tail logs or inspect a stuck record. Knowledge I want inline the moment I'm debugging, not a report.

## Why I don't build a "Go agent"

Because "Go" describes the material, not the task. The observability concern shows up as a *skill* when I'm writing the handler and could show up as an *agent* when I'm reviewing the diff — same topic, two shapes, two primitives. Folding both into one "Go agent" gets you a tool whose trigger overlaps three others and that you invoke at the wrong time half the time.

There's also a quiet cost to every extra agent: it isn't the writing, it's that it competes for your attention with the ones you already have. Skill [precedence and discoverability](/claude-code-skill-priority/) is a real constraint. The OpenTelemetry knowledge, written well as a skill so the instrumentation is right the first time, makes a separate "OTel auditor" agent almost redundant — I'd only add it if I wanted a deliberate second net on review, not for completeness.

## Impact

- **No more duplicated tooling.** Sorting by task shape collapses the "Go reviewer / Go tester / Go checker" sprawl into a small set where each thing has one job and one moment.
- **The right tool shows up on its own.** Skills auto-load while I write; the review agents are the deliberate pre-PR pass. I stopped asking "which one do I invoke now?"
- **Code is right earlier.** The expensive concerns (idioms, observability, the vendor dance) are inline skills, so they shape the code instead of arriving as a late verdict.

## Decisions

- **Organize by task shape, not by language.** Inline-while-writing → skill; isolated-parallel-verdict → agent. "Go" is never the deciding factor.
- **Default knowledge to a skill.** Reserve agents for investigation and review that genuinely benefit from an isolated, parallel context.
- **Add an agent only on proven, recurring need** — not to cover a topic a skill already handles inline.

## Limitations

- **A skill with a weak `description` is invisible.** If it doesn't auto-trigger on the right phrasing, the knowledge may as well not exist — the discoverability work is part of the cost.
- **An agent is the wrong shape for anything you needed inline.** A verdict that arrives after you've written the code can only criticize, not prevent.
- **Some cases are genuinely a judgment call.** Observability is the clearest one: skill, agent, or both is a deliberate trade-off, not a rule you can hard-code.

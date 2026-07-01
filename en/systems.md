---
layout: default
title: "Systems I've built"
description: "Real automations, tools and AI-augmented developer workflows I've shipped — not feature explorations."
permalink: /en/systems/
lang: en
nav: systems
systems_ref: true
---
<div class="container">
<article class="prose" markdown="1">
{%- assign _now = site.time | date: '%s' | plus: 0 -%}
{%- assign _pub = "" -%}
{%- for _p in site.posts -%}{%- if _p.lang == page.lang -%}{%- assign _pts = _p.date | date: '%s' | plus: 0 -%}{%- if _pts <= _now -%}{%- assign _pub = _pub | append: _p.url | append: "|" -%}{%- endif -%}{%- endif -%}{%- endfor -%}

# Systems I've built

I'm a backend engineer with AI woven through the daily workflow — debugging, code review, model and design decisions, agentic orchestration — and I build the systems that make those repeatable across a team: plugin marketplaces, multi-repo workspaces, review pipelines. All of it started inside a real engineering org with a real problem, not as a feature exploration.

Below is the working set — the practices I rely on and the systems I've shipped — grouped by what it is. Each link goes to a post that names the problem solved, the decisions made, and the real limitations.

## AI coding systems in practice

- **Per-team plugin marketplace** — one Git repo, one plugin folder per team, plus a shared `common` plugin promoted bottom-up. Replaces three incompatible ways of distributing Claude skills across teams. → [How a per-team plugin marketplace stopped Claude skill duplication across teams](/claude-code-team-marketplace/)

- **Multi-repo Claude workspace** — open Claude one level above your repos so a single context, a single memory folder and a single permissions scope cover ~25 services at once. → [Multi-repo workspace](/claude-code-multi-repo-workspace/)

- **Debugging a distributed bug with AI** — hand the model the trace, the real code path, and what you've ruled out, then ask for ranked hypotheses and the cheapest confirmation — not "fix it."{% if _pub contains "/debugging-with-ai-hand-it-the-evidence/" %} → [Debugging with AI](/debugging-with-ai-hand-it-the-evidence/){% endif %}

- **Designing a service with AI as interlocutor** — bring your own design and have the model attack it (where the boundary leaks, what breaks under partial failure), argue both sides of a trade-off, and keep the pen.{% if _pub contains "/designing-a-service-with-ai-as-interlocutor/" %} → [Designing a service](/designing-a-service-with-ai-as-interlocutor/){% endif %}

- **Model choice as an engineering decision** — match the tier and the effort dial to the task instead of defaulting to the biggest model; decide on a real eval set, not on instinct.{% if _pub contains "/picking-the-right-model-for-the-task/" %} → [Picking a model](/picking-the-right-model-for-the-task/){% endif %}

## Claude Code for real automation

- **Fork-session pattern** — branch the current Claude conversation to try an alternative without losing the in-flight state of the original. → [Fork a session](/claude-code-fork-session/)

- **The `!` shortcut as deterministic-input shortcut** — drop shell output into the conversation without going through the model when you need exact bytes, not a paraphrase. → [Bang shortcut](/claude-code-bang-shortcut/)

- **Scope vs. permissions, kept separate** — `cwd` decides what Claude *sees*, `.claude/settings.json` decides what Claude *does*. Treat them as two independent levers in every project. → [Scope vs. permissions](/claude-code-scope-vs-permissions/)

## Agentic workflows in production

- **Subagent-based exploration to keep main context lean** — push grep/read-heavy work into an `Explore` subagent so the main session pays ~1k for a summary instead of ~40k for a transcript. → [Subagents have their own context window](/claude-code-subagents/)

- **`/goal` loop for autonomous iteration** — stop typing "keep going" between turns; declare a condition and let Claude iterate until it's met. → [The `/goal` loop](/claude-code-goal/)

- **Persistent memory as cross-session state** — markdown files Claude writes on its own and reloads next time, indexed by `cwd`. Use it deliberately, prune it like a `.bashrc`. → [Persistent memory](/claude-code-persistent-memory/)

- **Multi-agent fan-out, costed honestly** — when N parallel agents beat one (independent, wide work) and when they just multiply the bill; decompose by genuine independence and budget the merge.{% if _pub contains "/multi-agent-orchestration-when-its-worth-it/" %} → [When the fan-out pays for itself](/multi-agent-orchestration-when-its-worth-it/){% endif %}

## Failure cases & lessons learned

- **Skill priority and silent overrides** — a personal skill silently wins over a project or plugin skill with the same name. Distinctive names, not clever overrides. → [Which skill wins when names collide?](/claude-code-skill-priority/)

- **The context window as a budget, not a capacity** — once compaction starts, the model is reasoning over a lossy summary. Plan the session so you don't get there by accident. → [Context window](/claude-code-context-window/)

- **Session rewind for cheap recovery** — small unit of undo on a Claude session so a bad turn doesn't poison the rest. → [Sessions rewind](/claude-code-sessions-rewind/)

- **Reviewing AI code for its failure modes** — AI code is *plausibly* wrong, not obviously wrong; a checklist for the seams it can't see — error paths, blast radius beyond the diff, concurrency, scope creep.{% if _pub contains "/reviewing-ai-generated-code-before-you-trust-it/" %} → [Reviewing AI-generated code](/reviewing-ai-generated-code-before-you-trust-it/){% endif %}

- **Green tests that prove nothing** — AI-written tests pass by asserting the mock or snapshotting the bug as expected; write the failing test first and read every assertion for can-it-fail.{% if _pub contains "/testing-with-ai-green-doesnt-mean-tested/" %} → [Testing with AI](/testing-with-ai-green-doesnt-mean-tested/){% endif %}

- **The day AI nearly shipped a bug to prod** — a clean diff, a green suite, and a confident explanation are one source agreeing with itself; the independent check has to come from outside the model's own loop.{% if _pub contains "/the-day-ai-nearly-shipped-a-bug-to-prod/" %} → [What I changed](/the-day-ai-nearly-shipped-a-bug-to-prod/){% endif %}

</article>
</div>

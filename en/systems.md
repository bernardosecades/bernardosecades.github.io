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

# Systems I've built

I build AI-augmented developer systems: plugin marketplaces, multi-repo workspaces, review pipelines, agentic workflows. Each one started inside a real engineering org with a real problem, not as a feature exploration.

Below is the working set, grouped by what kind of system it is. Each link goes to a post that names the problem solved, the technical decisions made, and the real limitations.

## AI coding systems in practice

- **Per-team plugin marketplace** — one Git repo, one plugin folder per team, plus a shared `common` plugin promoted bottom-up. Replaces three incompatible ways of distributing Claude skills across teams. → [How a per-team plugin marketplace stopped Claude skill duplication across teams](/claude-code-team-marketplace/)

- **Multi-repo Claude workspace** — open Claude one level above your repos so a single context, a single memory folder and a single permissions scope cover ~25 services at once. → [Multi-repo workspace](/claude-code-multi-repo-workspace/)

## Claude Code for real automation

- **Fork-session pattern** — branch the current Claude conversation to try an alternative without losing the in-flight state of the original. → [Fork a session](/claude-code-fork-session/)

- **The `!` shortcut as deterministic-input shortcut** — drop shell output into the conversation without going through the model when you need exact bytes, not a paraphrase. → [Bang shortcut](/claude-code-bang-shortcut/)

- **Scope vs. permissions, kept separate** — `cwd` decides what Claude *sees*, `.claude/settings.json` decides what Claude *does*. Treat them as two independent levers in every project. → [Scope vs. permissions](/claude-code-scope-vs-permissions/)

## Agentic workflows in production

- **Subagent-based exploration to keep main context lean** — push grep/read-heavy work into an `Explore` subagent so the main session pays ~1k for a summary instead of ~40k for a transcript. → [Subagents have their own context window](/claude-code-subagents/)

- **`/goal` loop for autonomous iteration** — stop typing "keep going" between turns; declare a condition and let Claude iterate until it's met. → [The `/goal` loop](/claude-code-goal/)

- **Persistent memory as cross-session state** — markdown files Claude writes on its own and reloads next time, indexed by `cwd`. Use it deliberately, prune it like a `.bashrc`. → [Persistent memory](/claude-code-persistent-memory/)

## Failure cases & lessons learned

- **Skill priority and silent overrides** — a personal skill silently wins over a project or plugin skill with the same name. Distinctive names, not clever overrides. → [Which skill wins when names collide?](/claude-code-skill-priority/)

- **The context window as a budget, not a capacity** — once compaction starts, the model is reasoning over a lossy summary. Plan the session so you don't get there by accident. → [Context window](/claude-code-context-window/)

- **Session rewind for cheap recovery** — small unit of undo on a Claude session so a bad turn doesn't poison the rest. → [Sessions rewind](/claude-code-sessions-rewind/)

</article>
</div>

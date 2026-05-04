---
title: "Claude Code: one workspace for many repos"
date: 2026-05-05 00:00:00
lang: en
ref: claude-code-multi-repo-workspace
tags: [claude-code, workflow]
read_min: 3
excerpt_text: "Launch Claude in the parent of your repos so cross-service tickets stop being N isolated sessions — and let a workspace-level CLAUDE.md describe how the pieces connect."
---

In a monolith you only ever work in one folder, so there's nothing to think about. In a multi-repo setup it's different: a single ticket often touches a microservice and a shared library that lives in a sibling folder, sometimes used by other services too.

My layout looks like this:

```text
~/work/
├── microservice-a/
├── microservice-b/
├── library-core/
└── library-x/
```

## 1. Launch Claude from the parent

```bash
cd ~/work
claude
```

That's it. Every subfolder under `~/work/` is in scope — no need to "open" or "add" anything per microservice. That doesn't mean Claude reads or indexes them automatically at launch: it only touches a file when you mention it (`@path`) or when the model decides to read it during the task. If a ticket touches `microservice-a` and `library-core`, both are available for when they're needed.

The `--add-dir` flag only enters the picture when you need a folder **outside** your working directory:

```bash
claude --add-dir /some/external/path
```

## 2. `@path` doesn't widen scope — it forces a read

This is where I see people get stuck. `@microservice-a/src/foo.ts` does **not** change permissions or scope: what it does is force a read of that file and include it in context right now. Same for `@microservice-a/` to inline a directory listing.

Put another way:

- The working directory (and `--add-dir`) decides **what Claude can read**.
- `@path` decides **what Claude reads right now**, without waiting for the model to go fetch it.

## 3. Stack CLAUDE.md by scope

The two-level layout that works for me:

- **`~/work/CLAUDE.md`** — the workspace map. What each folder is, which service uses which library, shared contracts, naming. It's the `CLAUDE.md` of the directory where you launched Claude, so it lands in the initial context.
- **`~/work/<service>/CLAUDE.md`** — service-specific notes (build commands, test setup, quirks). These live in subdirectories, so Claude tends to pull them in when it starts working inside that service. It's not deterministic step by step — it depends on how Claude navigates during the task.

The payoff: the initial context stays light (only the workspace map), and the service-specific stuff appears when the work actually lands there.

`~/.claude/CLAUDE.md` still applies on top of all of this for personal preferences across every project.

## Why I do this

A cross-service ticket stops being N isolated sessions where I have to re-explain how the pieces fit together. The dependencies between services live explicitly in the workspace `CLAUDE.md`, and Claude pulls in each service's own rules only when the work actually lands there.

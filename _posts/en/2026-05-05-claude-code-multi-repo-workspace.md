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

That's it. Every subfolder under `~/work/` is accessible to Claude — no need to "open" or "add" anything per microservice. If a ticket touches `microservice-a` and `library-core`, both are already in scope.

The `--add-dir` flag only enters the picture when you need a folder **outside** your working directory:

```bash
claude --add-dir /some/external/path
```

## 2. `@path` is a reference, not a permission

This is where I see people get stuck. Writing `@microservice-a/src/foo.ts` in your prompt does **not** grant Claude access to that file — it just inlines its content into the conversation as a shortcut. Same for `@microservice-a/` to inline a directory listing.

Access is decided by the working directory (and `--add-dir`). `@` is just a way to pull something in faster than asking Claude to go read it.

## 3. Stack CLAUDE.md by scope

The two-level layout that works for me:

- **`~/work/CLAUDE.md`** — the workspace map. What each folder is, which service uses which library, shared contracts, naming. Loaded at launch because it sits at the working directory.
- **`~/work/<service>/CLAUDE.md`** — service-specific notes (build commands, test setup, quirks). These live in subdirectories, so they load **on demand** the first time Claude reads a file in that service.

The payoff: the initial context stays light (only the workspace map), and the service-specific stuff loads only when Claude actually goes there.

`~/.claude/CLAUDE.md` still applies on top of all of this for personal preferences across every project.

## Why I do this

A cross-service ticket stops being N isolated sessions where I have to re-explain how the pieces fit together. The dependencies between services live explicitly in the workspace `CLAUDE.md`, and Claude pulls in each service's own rules only when the work actually lands there.

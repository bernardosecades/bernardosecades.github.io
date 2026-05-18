---
title: "Claude Code: the persistent memory you didn't know you had"
date: 2026-05-21 00:00:00
lang: en
ref: claude-code-persistent-memory
tags: [claude-code, memory, workflow]
read_min: 4
excerpt_text: "Claude keeps notes about you between sessions. Where they live, what gets saved, the doubts everyone has, and how to see, prune, and steer them."
---

The [previous post on /goal](/claude-code-goal/) was about state **within** a session — Claude iterating until a condition is met. This one is about state **between** sessions: the small set of notes Claude writes on its own and reloads next time you open the project.

Most people are surprised it exists. Then surprised again when they realize it's just markdown files on their own disk.

## What persistent memory is — the key

It's a folder under `~/.claude/projects/<slug-of-your-cwd>/memory/` with:

- A `MEMORY.md` **index** — one line per memory, always loaded at the start of every session.
- Individual `.md` files — one per memory, each with frontmatter and a short body. Loaded **on demand** when a memory looks relevant.

Claude writes there itself, without being asked, when it learns something worth remembering. You can also tell it *"remember that…"* or *"forget X"*.

This is **not** `CLAUDE.md`. `CLAUDE.md` is documentation you write for Claude (architecture, conventions, commands). Persistent memory is notes **Claude writes for itself** about you, your preferences, and the work in flight.

## Where it lives

```text
~/.claude/projects/
└── -home-you-projects-orders/        ← one folder per cwd
    └── memory/
        ├── MEMORY.md                 ← the index, always loaded
        ├── user_role.md
        ├── feedback_terse_replies.md
        ├── project_auditlib_rewrite.md
        └── reference_linear_ingest.md
```

The folder name is the absolute path of the cwd, with `/` turned into `-`. One project, one folder. Open Claude in a different directory and you get a different memory set — they don't bleed across.

## The four types

Each file declares a `type:` in its frontmatter:

- **user** — who you are, your role, what you know. *"Senior Go dev, new to the React side of this repo."*
- **feedback** — how you want to be worked with. *"Prefers terse replies, no end-of-turn summaries."*
- **project** — what's going on right now in the work. *"The `auditlib` rewrite is driven by legal/compliance requirements, not tech-debt cleanup. Why: a legal review flagged session token storage. How to apply: favor compliance correctness over ergonomics."*
- **reference** — where to look in external systems. *"Pipeline bugs are tracked in Linear project `INGEST`."*

The four types map to the four questions Claude tends to ask itself when it picks up a session: *who am I working with, how do they want me to work, what's in flight, where do I look for more.*

## Doubts I keep hearing

**Does it sync across machines?** No. Plain files under `~/.claude/`. Move them yourself with `rsync` if you want.

**Is it shared across projects?** No. One folder per cwd. If you open Claude in `~/projects/orders` vs `~/projects/billing`, the two memory sets are independent.

**Does Claude read everything every turn?** No. Only `MEMORY.md` (the index, truncated past 200 lines) loads automatically. Individual files load when the description in the index suggests they're relevant.

**What if something embarrassing or secret gets saved?** They're plain markdown. `cat`, `vim`, `rm` — same as any file. Nothing is encrypted, nothing is uploaded anywhere by the memory subsystem itself.

**Does it expire?** Not on its own. `project` memories age fast (a deadline passes, a refactor lands), but the file stays until someone — you or Claude — removes it. Worth a glance every few weeks.

## Tricks to see, manage, and optimize

**See what's there.** Two ways:

```bash
ls ~/.claude/projects/-home-you-projects-orders/memory/
cat ~/.claude/projects/-home-you-projects-orders/memory/MEMORY.md
```

Or just ask: *"what do you remember about me on this project?"* Claude will read the index and walk you through it.

**Force a save.** *"Remember that I prefer Postgres over MySQL for new services because of JSONB and partial indexes."* Claude picks a type, writes the file, updates the index.

**Force a forget.** *"Forget the memory about the `auditlib` rewrite being driven by legal — that's wrong now."* Claude finds the file, deletes it, prunes the index line. If it can't find what you mean, open the folder and delete by hand — it's just files.

**Prune project memories regularly.** They have a `Why:` line by design. Skim them: if the *why* is no longer load-bearing (deadline passed, decision reversed, person left), delete. Outdated `project` memories will confidently mislead the next session.

**Keep the index lean.** `MEMORY.md` truncates past 200 lines. One memory = one line, under ~150 chars: `- [Title](file.md) — one-line hook`. If yours is growing past that, consolidate duplicates and drop dead ones.

**Edit by hand when faster.** The files are markdown with frontmatter. If you spot something wrong, fix it directly. Two gotchas: keep `name:` in the file matching the index entry, and keep `type:` valid (`user`, `feedback`, `project`, `reference`).

**Working across sibling repos?** Memory is keyed by cwd. If you open Claude in `~/projects/orders` and `~/projects/billing` separately, neither knows what the other learned. The [multi-repo workspace setup](/claude-code-multi-repo-workspace/) sidesteps this: open Claude one level up, and the memory folder is shared across all the repos beneath.

## Why this matters

The memory doesn't make Claude smarter. It saves you from re-explaining yourself every Monday — the project's quirks, your preferences, what's in flight, where things live.

But it's mutable state that you don't actively maintain. What Claude wrote two months ago can mislead you today. Read it once in a while. Prune it like you'd prune a `.bashrc` you forgot existed. The [context window](/claude-code-context-window/) is the budget for one session; persistent memory is the budget across all of them — both deserve the same care.

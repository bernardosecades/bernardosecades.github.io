---
title: "Claude Code: which skill wins when names collide?"
date: 2026-05-11 00:00:00
lang: en
ref: claude-code-skill-priority
tags: [claude-code, skills, workflow]
read_min: 2
excerpt_text: "Clone a repo that ships its own skills and suddenly you have two skills with the same name. There's a clear priority order that decides which one runs."
---

Clone a repository that ships its own skills and suddenly you have two skills with the same name. Which one runs? Claude Code has a fixed priority order:

1. **Enterprise** * — managed settings, highest priority
2. **Personal** — your home directory (`~/.claude/skills`)
3. **Project** — the `.claude/skills` directory inside a repository
4. **Plugins** — installed plugins, lowest priority

So if your company ships an enterprise `code-review` skill and you have a personal `code-review` skill, the enterprise version always wins.

> (*) Enterprise skills apply when you use Claude Code with a **corporate account** — typically when your organization manages Claude through Anthropic's enterprise offering and pushes settings centrally. If you're on a personal account, there are no enterprise skills in the stack.

## Concrete example

You have:

```
~/.claude/skills/code-review       ← Personal
repo/.claude/skills/code-review    ← Project
```

When Claude tries to run `code-review`:

```
Check Enterprise → not found
Check Personal   → found ✔ STOP
                   never reaches Project
```

**Result:** your personal skill runs. The repo's skill is completely ignored.

## What this means in practice

Personal skills override project skills. That's useful when you want to add your own shortcuts on top of what a repo provides, without touching the repo itself.

Project skills override plugins. Repos can ship opinionated workflows that take precedence over anything you installed from the marketplace.

The enterprise layer exists so organizations can enforce standards that individuals can't accidentally override — even if someone adds a personal or project skill with the same name.

## Avoiding collisions

Use descriptive names instead of generic ones. `review` is a collision waiting to happen. `frontend-review`, `backend-review`, or `api-review` are specific enough that the chance of an accidental name match drops to near zero.

The priority order is a fallback, not an invitation to rely on it. Clear names are better than clever overrides.

---
title: "Claude Code: name a session and fork it for an alternative path"
date: 2026-05-04 10:00:00
lang: en
ref: claude-code-fork-session
tags: [claude-code, workflow]
read_min: 2
excerpt_text: "Rename a Claude Code session with /rename, then fork it into an independent copy to try a different approach without losing the original."
---

A small two-step workflow I use whenever I want to try a different approach in a session without losing the original thread. The idea is very close to a `git branch`: same starting point, separate branch.

## 1. Name the session

Inside the session, give it a memorable name:

```text
/rename main-context
```

Now you can refer to the session by name instead of by its raw ID.

## 2. Fork it from outside

From your terminal, resume that named session and fork it:

```bash
claude --resume main-context --fork-session
```

This copies the conversation history into a **new session** and drops you into it. The original `main-context` stays untouched — you can keep coming back to it as your "stable" thread.

## Why I do this

- I treat one named session as my main working context, and fork it whenever I want to explore a side path.
- If the side path doesn't pan out, I just resume the original and try again from a clean state.

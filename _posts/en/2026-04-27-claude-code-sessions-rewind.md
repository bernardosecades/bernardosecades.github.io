---
title: "Claude Code: continue sessions and rewind changes"
date: 2026-04-27 11:00:00
lang: en
ref: claude-code-sessions-rewind
tags: [claude-code, workflow]
read_min: 4
excerpt_text: "Resume previous Claude Code sessions, branch from any point, and roll back unwanted edits with a single command."
---

Three small Claude Code tricks I use every day.

## Continue the last session

Picks up exactly where you left off — same context, same history.

```bash
claude --continue
# or the short form
claude -c
```

## Resume an older session

Opens a picker with your past sessions (from days or weeks ago) so you can jump back into any of them.

```bash
claude --resume
# or
claude -r
```

## Rewind changes

If Claude went down the wrong path, press **Esc twice** (`Esc Esc`) to rewind the conversation to an earlier point. You can also use the `/rewind` slash command. This undoes the messages *and* the file edits made since then, so you don't have to clean up by hand.

That's it — three shortcuts that save a lot of time.

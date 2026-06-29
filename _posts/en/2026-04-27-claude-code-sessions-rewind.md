---
title: "Claude Code: continue sessions and rewind changes"
date: 2026-04-27 11:00:00
lang: en
ref: claude-code-sessions-rewind
tags: [claude-code, workflow]
read_min: 4
excerpt_text: "Made a mess in Claude Code? Use /rewind to roll back edits and conversation step by step, then resume any past session right where you left off."
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

Opens a picker with your past sessions (from days or weeks ago) so you can jump back into any of them. The picker matches on the session name too, so if you [name sessions after the Jira ticket](/claude-code-sessions-jira-tickets/) you're working on, the right one is a keystroke away.

```bash
claude --resume
# or
claude -r
```

## Rewind changes

If Claude went down the wrong path, press **Esc twice** (`Esc Esc`) to rewind the conversation to an earlier point. You can also use the `/rewind` slash command. This undoes the messages *and* the file edits made since then, so you don't have to clean up by hand. Rewinding rolls *back* to an earlier point; when you'd rather keep the current thread intact and try an alternative in parallel, [fork the session](/claude-code-fork-session/) instead.

That's it — three shortcuts that save a lot of time.

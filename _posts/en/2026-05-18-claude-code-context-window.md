---
title: "Claude Code: keeping the context window healthy"
date: 2026-05-18 00:00:00
lang: en
ref: claude-code-context-window
tags: [claude-code, context, workflow]
read_min: 4
excerpt_text: "The context window is a limited resource. Four habits I use daily to keep it healthy: /clear and /compact, subagents, @path, and plan mode."
---

The Claude Code context window is finite. Every message, every tool call, every file read takes a slice of it. When it fills up, two things happen: the model starts forgetting what was said early in the session, and per-turn costs jump because processing 200k tokens each round isn't free.

Four habits I use daily to keep it healthy.

## /clear, /compact and auto-compact

Three ways to free up context, each for a different situation.

`/clear` wipes everything and starts a fresh session. I use it when switching tasks: the payments bug is done, now I'm touching logs, I don't need the conversation dragging the previous thread along.

`/compact` asks the model to summarize the context and keeps only that summary. Useful after an hour of work, when the 12 files I read at the start are no longer pulling their weight but I do want to keep the decisions we made along the way.

Auto-compact fires on its own when you hit ~95% of the window. It works, but it can catch you mid-task: the automatic summary may drop nuances you were about to lean on. If I see the indicator climbing past 70% and the task isn't closed, I trigger `/compact` myself before the model decides for me.

## Subagents to keep the main session clean

When a task starts with *"I need to understand how X talks to Y"*, that exploration phase easily burns 30-50k tokens: greps, finds, reading several files to reconstruct the flow.

Instead of doing that in my main session, I launch an `Explore` subagent. The subagent has **its own window**: it spends its tokens exploring and returns a half-kilobyte summary. My session only receives the summary, not the dozen full files.

I start the task with the main context almost empty, already holding the mental map I needed, and the window stays free for the real work — writing, reviewing, iterating.

## @path: hand over the file, don't ask Claude to find it

Two ways to get a file into context: ask Claude *"read the tests to figure out what X does"* (Claude lists, opens, sometimes picks the wrong file), or type `@path/to/test.go` yourself.

The second is direct. The file is injected and that's it. No searches, no intermediate tool calls, no back-and-forth.

Same idea with the [`!` shortcut](/claude-code-bang-shortcut/): `!cat config.yaml` drops the command output into context without going through the model at all.

Claude **searching and reading ten files** to find the right one costs more than just handing over the two that matter from the start.

## Plan mode and when to start a new session

Plan mode (Shift+Tab twice) makes Claude design without executing. No tool calls, no long outputs, no 800-line logs dumped into the conversation. Just reasoning and a plan at the end. The design phase stays cheap, and once you execute you already know what to touch.

To decide between continuing and starting fresh:

- **New task, same project** → `/clear`. Keep CLAUDE.md and the local setup, drop the conversation.
- **Want to explore an alternative without losing the current state** → [fork the session](/claude-code-fork-session/). Two branches in parallel.
- **Claude went down the wrong path this session** → [rewind to an earlier point](/claude-code-sessions-rewind/). Undoes the messages *and* the file edits since then, no manual cleanup.
- **Completely different task** → new session from scratch.

Starting clean has way less friction than fighting a bloated session.

## Why this matters

The window is a **budget**, not an infinite. Every read, every tool call, every subagent has a cost, and answer quality drops as you approach the ceiling.

If you notice late, `/compact` bails you out. If you notice in time, you don't even need it.

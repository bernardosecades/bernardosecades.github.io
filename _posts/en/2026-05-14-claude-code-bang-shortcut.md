---
title: "Claude Code: the ! shortcut for shell commands"
date: 2026-05-14 00:00:00
lang: en
ref: claude-code-bang-shortcut
tags: [claude-code, shell, workflow]
read_min: 2
excerpt_text: "Prefix a command with ! and it runs directly in your shell, bypassing the model entirely. Faster, and no tokens spent on a round-trip you don't need."
---

`!` is a shortcut to run shell commands directly from the Claude Code prompt, without asking Claude to do it for you.

## How to use it

Prefix the command with `!`:

```
!go test ./... --tags unit,integration
!go build ./...
!git status
!ls internal/
```

<video controls playsinline>
  <source src="/assets/videos/demo-shell-command.webm" type="video/webm">
</video>

The command runs in your shell and its output lands in the conversation. Claude can read it and use it as context for the next steps.

## Why it saves tokens

When you type `!go test ./... --tags unit,integration` directly, the command runs without touching the model. No input tokens, no reasoning tokens, no tool call tokens.

Compare that to: *"Claude, run the tests please."* In that case the model:

1. Processes your message (input tokens)
2. Reasons about which tool to use (output tokens)
3. Makes the Bash tool call (tool use tokens)
4. Receives the result (input tokens)
5. Replies with something like *"the tests passed"* (output tokens)

With `!go test ./... --tags unit,integration` you skip all of that. The command output is injected into context, but nothing more — no model round-trip.

## When to use it

If you already know exactly which command to run, `!` is the right call. It's faster than describing what you want and waiting for Claude to decide which tool to invoke.

If you don't know the exact command, or you need Claude to interpret the output and act on it, asking Claude directly makes more sense.

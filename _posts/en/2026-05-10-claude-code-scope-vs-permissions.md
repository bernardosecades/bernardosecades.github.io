---
title: "Claude Code: scope vs permissions"
date: 2026-05-10 00:00:00
lang: en
ref: claude-code-scope-vs-permissions
tags: [claude-code, settings, permissions, workflow]
read_min: 3
excerpt_text: "The working directory decides what Claude can read. Actions — running commands, writing files, calling MCP tools — are governed by .claude/settings.json. Two different layers, easy to confuse."
---

The [previous post on multi-repo workspaces](/claude-code-multi-repo-workspace/) framed the working directory as what Claude can see. That's only half of the picture: it covers **reads**. The other half — **actions** like running a Bash command, writing a file, or calling an MCP tool — is governed by something else: `.claude/settings.json`.

Two separate layers. Confusing them is what makes people ask *"the folder is in scope, why is Claude still asking before running `rm`?"*, or the opposite, *"I added the path and now it's running migrations on its own."*

## Reads vs actions

- **Working directory** (and `--add-dir`) → what Claude is *allowed to see*.
- **`.claude/settings.json`** → what Claude is *allowed to do*.

A file inside the working directory is technically reachable. Whether Claude can *write* to it, or run a command that touches it, still goes through the rules in `settings.json`.

## Where settings.json lives

The file layers across several locations:

- **User** — `~/.claude/settings.json`. Personal defaults across every project.
- **Project (shared)** — `<repo>/.claude/settings.json`. Committed to the repo, applies to anyone working on it.
- **Project (local)** — `<repo>/.claude/settings.local.json`. Not committed (gitignored by default), for your own project-specific tweaks.
- **Enterprise managed** — pushed centrally by your organization, top priority.

These layers stack. A `deny` rule always wins, so a project-level deny overrides a user-level allow.

## A minimal example

```json
{
  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(go test:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Read(./.env)",
      "Edit(./prod-config.yaml)"
    ]
  }
}
```

`allow` runs without prompting. `deny` is hard-blocked. Anything not matched falls into the default flow (Claude asks).

<video controls playsinline>
  <source src="/assets/videos/demo-settings-permissions.webm" type="video/webm">
</video>

In the demo Claude refuses to read `.env` because of the `deny` rule. If you actually want that content in context, the [`!` shortcut](/claude-code-bang-shortcut/) is the escape hatch: `! cat .env` runs in your shell and the output lands in the conversation. The `deny` applied to Claude's autonomous read; you typing the command yourself is a separate decision.

## Why this matters

The `Read(./.env)` rule above is the clearest illustration: the file lives inside the working directory, so by the read-scope rule alone it would be visible. The `deny` in `settings.json` overrides that. Scope says *"you could reach this"*; `settings.json` says *"no, you can't"*.

Same with Bash. The folder being in scope tells Claude *where* commands run. `settings.json` tells it *which* commands are pre-approved.

If you find yourself approving `Bash(go test:*)` ten times a day, that's the signal to add it to `allow`. If there's a command you never want Claude to run on its own, that's the signal to add it to `deny`.

The mental model that works for me: the working directory is the *map* of where Claude can move; `settings.json` is the *rulebook* of what it's allowed to do once it gets there.

---
title: "Claude Code: a /worktree slash command that bootstraps a Go worktree and opens GoLand"
date: 2026-06-30 00:00:00
lang: en
ref: claude-code-worktree-command
tags: [claude-code, slash-commands, worktree, golang]
read_min: 4
excerpt_text: "The built-in `claude -w` drops you into a terminal inside a fresh worktree — perfect, until your day actually lives in GoLand and you keep redoing the same Go bootstrap by hand. A custom /worktree command encodes that setup once: one ticket ID in, a branch, a copied .env, and GoLand open on it."
---

In the [previous post](/claude-code-git-worktree-parallel/) I landed on `claude -w ORD-2231` as the day-to-day way to run two tasks on the same service without the stash + checkout ritual. It creates the worktree, branches off clean `origin/HEAD`, starts a session inside, and even cleans up after itself. For a quick fix it's exactly right.

But my day doesn't live in a terminal — it lives in GoLand. And `-w` drops me into a terminal session, not my IDE. So every time I spin up a worktree the real flow has three more manual steps stapled to the end: copy the `.env` the service needs, make sure modules are there, and open the thing in GoLand. Three steps I do from memory, slightly differently each time, and occasionally forget the first one of (the service won't start, cue five minutes of confusion — exactly Gotcha 1 from last time).

That's the shape of work worth turning into a command: short, repetitive, and you keep doing it by hand.

## What the built-in leaves on the table

`claude -w` is `git worktree add` plus conventions (path under `.claude/worktrees/`, branch `worktree-<name>`, base `origin/HEAD`) plus a lifecycle, plus a Claude session in a terminal. That's the right default and I still reach for it first.

What it doesn't do — by design, because it can't know your stack — is the team-specific bootstrap:

- **Open your IDE on the worktree.** A terminal session isn't where I read Go code; GoLand is.
- **Your own branch and path convention.** I want the branch named `ORD-2231`, plain, and a `.trees/` folder — not `worktree-ORD-2231` under `.claude/`.
- **Copy the git-ignored config the service needs to boot.** The `.env` that doesn't travel with the worktree.

None of that is hard. It's just three commands I'd rather not retype — so I encode them once.

## A command is just a prompt

A Claude Code slash command is a markdown file in `.claude/commands/`. The filename is the command name; the body is the prompt Claude runs; `$ARGUMENTS` (or `$1`, `$2`, …) is where the arguments land. A little frontmatter makes it discoverable and safe:

```markdown
---
description: Create a Go worktree for a Jira ticket and open it in GoLand
argument-hint: <TICKET-ID>
allowed-tools: Bash(git worktree:*), Bash(cp:*), Bash(goland:*), Bash(test:*)
---

Create a git worktree for ticket `$1`, set it up for Go, and open it in GoLand.

Steps:
1. If `.trees/$1` already exists, stop and tell me it's already there — don't recreate it.
2. Create the worktree on a new branch `$1` from `origin/main`:
   git worktree add -b $1 .trees/$1 origin/main
3. Copy the git-ignored local config the service needs (`.env`, `.env.local`) from
   the repo root into `.trees/$1`. Go's module cache is global, so there's nothing
   else to link.
4. Open the worktree in a new GoLand window: goland .trees/$1
5. Print the branch name and path, and remind me the runtime (ports, DB) is shared.
```

Save it as `.claude/commands/worktree.md` and it becomes `/worktree <TICKET-ID>`. The `argument-hint` shows up in the autocomplete; `allowed-tools` scopes what the command may run without asking, so a `/worktree` invocation isn't a blank cheque on your shell.

Note step 1: this is a *prompt*, not a shell script, so I just tell it "if the worktree exists, stop." Claude checks and bails with a clear message instead of fighting git's `fatal: ... already used by worktree` error.

## Why the Go setup is this short

Dependencies are a non-problem. Go's module cache is **global** (`$GOPATH/pkg/mod`, with builds cached in `$GOCACHE`): every worktree on the machine already shares it, so the first `go build` in a new worktree resolves from cache, not from the network. There's nothing to copy and nothing to link — the worktree is buildable the moment git creates it.

What does need handling is the one thing the previous post flagged as Gotcha 1: the files git ignores but the service needs to boot. In a Go service that's almost always the `.env`. So step 3 is a plain copy — the single manual step that, forgotten, makes the service die on startup. And step 4 works because JetBrains ships a command-line launcher: `goland .trees/ORD-2231` opens that directory in a new GoLand window.

## Running it

From inside any session on the repo:

```text
> /worktree ORD-2287

✓ .trees/ORD-2287 doesn't exist yet
✓ git worktree add -b ORD-2287 .trees/ORD-2287 origin/main
✓ copied .env → .trees/ORD-2287/.env
✓ goland .trees/ORD-2287   (opening…)

Branch ORD-2287 ready at .trees/ORD-2287.
Heads up: ports and the local DB are still shared with your other worktrees.
```

One line in, a branch named after the ticket, a runnable checkout, and GoLand opening on it. The ticket ID that already named the [session and the branch](/claude-code-sessions-jira-tickets/) now names the worktree path too.

## Impact

- One line — `/worktree ORD-2287` — goes from ticket ID to a runnable Go worktree with GoLand open on it.
- The bootstrap is **encoded, not remembered**: the `.env` copy that I used to forget is now step 3, every time.
- The same ID names the branch, the path and the worktree — no new vocabulary to invent per task.

## Decisions

- **`-w` for quick terminal work, `/worktree` for the IDE flow.** The built-in is still the default; the command is the team-specific 20% it can't cover.
- **Branch named `<ticket>`, path `.trees/`.** My conventions, not the built-in's `worktree-<name>` under `.claude/`. The bare ticket ID is already unique and already names the session — no `feature/` prefix needed. `.trees/` goes in `.gitignore`.
- **Copy `.env`, don't symlink anything.** Go's global module cache means dependencies are already shared across worktrees — the only thing to carry over is the git-ignored config.

## Limitations

- **It's a convenience wrapper you own.** Unlike `-w`, nobody maintains it but you — if your bootstrap grows, the command grows with it.
- **`goland` has to be on your PATH.** Install the launcher from JetBrains Toolbox (or *Tools → Create Command-line Launcher*) or step 4 just fails.
- **The runtime is still singular.** This copies files; it doesn't isolate ports or databases. Everything in Gotcha 2 of the [worktree post](/claude-code-git-worktree-parallel/) still applies.
- **A prompt has judgment, a script has guarantees.** Claude executes the steps with reasoning, which is what lets step 1 bail gracefully — but if you need the exact same bytes every time, a shell script behind `claude -w` is the more deterministic tool.

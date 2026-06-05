---
title: "Claude Code: two tasks on the same service, two sessions, zero stash — git worktree"
date: 2026-06-16 00:00:00
lang: en
ref: claude-code-git-worktree-parallel
tags: [claude-code, git, worktree, workflow]
read_min: 5
excerpt_text: "You're mid-refactor and an urgent hotfix lands on the same service. The stash + checkout ritual now has a new casualty: the Claude session that knew your half-finished code. git worktree — in git since 2015, not a Claude Code thing — gives you a second working directory with its own session, without touching the first."
---

Tuesday, 11:40. I'm half a morning deep into ORD-2210, a refactor of the payment handler in the orders service: fifteen files touched, half of them not compiling, and a Claude session that knows every half-made decision. The incident fires: ORD-2231, the cancellation webhook is returning 500s in production. Same service. Drop everything.

The classic ritual: `git stash`, checkout main, fix, PR, come back, `git stash pop`, pray. But the ritual now has a casualty it didn't use to have: the Claude session. The entire conversation — the plan, the fifteen files, the why behind each change — describes a working tree that just stopped existing. Keep talking to it after the checkout and it reasons about code that's no longer on disk. The alternative, closing it and opening a fresh one, throws away the context you spent half a morning building.

The problem isn't git: it's that a clone has one working directory and you have two tasks. The solution is also git, and it isn't new. `git worktree` is not a Claude Code feature — I've met more than one developer convinced it is — it shipped in git 2.5, July 2015. What changed is that each working directory can now carry its own AI session on top, and suddenly there's a compelling reason to use it.

## One .git, several working directories

A worktree is a second working directory hanging off the **same** `.git`. It's not a clone: no second fetch, no second history. Objects, branches, stash, remotes and config are all shared; a commit made in one worktree is instantly visible in the other.

```bash
git worktree add -b hotfix/ORD-2231 ../orders-hotfix origin/main
```

```text
$ git worktree list
~/work/orders          3f2a91c [feature/ORD-2210-payment-handler]
~/work/orders-hotfix   8c17d04 [hotfix/ORD-2231]
```

The parallel flow falls out naturally:

```bash
cd ../orders-hotfix
claude -n ORD-2231
```

Second terminal, second session, [named after its ticket](/claude-code-sessions-jira-tickets/) as always. The feature session stays open in the first terminal, its working tree intact underneath. Switching tasks means switching windows — no stash, no checkout, no context to rebuild in either direction.

## The built-in shortcut

Claude Code packages exactly that flow into one flag:

```bash
claude -w ORD-2231
```

It creates the worktree under `.claude/worktrees/ORD-2231/`, gives it the branch `worktree-ORD-2231` and starts the session inside it. Three deliberate details:

- **The branch starts from `origin/HEAD`** — the remote default — not from your current branch. Exactly what a hotfix needs: branch off clean main, not off your half-finished feature. (If you want to branch from your local HEAD instead: `"worktree": {"baseRef": "head"}` in settings.)
- **It cleans up after itself.** When the session ends, if the worktree has no changes and no new commits, it's removed along with its branch. If it does, Claude asks whether to keep it.
- **`--resume` returns to the worktree.** The session is linked to it and reopens inside. In the picker, `Ctrl+W` shows sessions across all worktrees of the repo.

The worktree name and the session name are separate things; to keep the one-session-per-ticket convention, the two flags compose: `claude -w ORD-2231 -n ORD-2231`. And one variant that deserves its own line: `claude -w "#4812"` fetches that PR and builds the worktree on top of it — test a teammate's PR without touching your branch.

## Gotcha 1: what's shared and what isn't

The number one source of confusion. The `.git` is shared in full — branches, stash, objects, remotes, hooks, config. What does **not** travel is everything git doesn't version:

- `.env`, `.env.local`, `docker-compose.override.yml`, local certificates
- `vendor/`, `node_modules/`, build caches

The new worktree is born "clean": versioned files only. The service that started on the first try in your usual directory dies here asking for a `.env` that doesn't exist. It's the classic source of "why won't it start?" — and of the mistaken conclusion that worktrees "don't work".

Claude Code ships a fix: a `.worktreeinclude` file at the repo root, with `.gitignore`-style patterns:

```text
# .worktreeinclude
.env
.env.local
docker-compose.override.yml
```

Every file that matches (and is git-ignored) gets copied automatically into each new worktree. Commit it and the whole team inherits the fix. For manual worktrees, the equivalent is a `cp` in your bootstrap script — or knowing that the first time, you copy the `.env` by hand.

## Gotcha 2: the code is isolated; the runtime isn't

Two worktrees of the same microservice are two copies of the code and **zero** copies of the runtime. Both want port 8080, both want to bring up the same docker-compose, both point at the same local database. The first `make run` in the hotfix worktree can stomp on — or corrupt — whatever you had in flight in the feature one.

What works for me:

- **The runtime has a single owner.** The long-lived worktree runs the full stack; the hotfix one lives off tests — `make test-unit`, integration via testcontainers or ephemeral ports. For a hotfix that's almost always enough, and it's the zero-conflict option.
- **If you genuinely need two stacks:** compose already separates projects by directory name (or force your own with `docker compose -p ord-2231`), but *published* ports still collide — a `PORT=8081` override or a different `.env` per worktree. The fact that `.env` doesn't get shared automatically stops being a bug and becomes the feature.

## Impact

- The hotfix never touches the feature: zero stash, zero checkout, and no session reasoning about code that's no longer on disk.
- Switching tasks goes from a stateful ritual (stash, checkout, pop, conflicts) to switching terminals.
- With `-w`, the second task starts in seconds from clean `origin/HEAD` — and picks up after itself if it left nothing behind.
- It composes with [ticket-named sessions](/claude-code-sessions-jira-tickets/): the same ID names the branch, the PR, the session and now the worktree.

## Technical decisions

- **Manual to understand it, `-w` for the day-to-day.** `claude -w` isn't magic: it's `git worktree add` plus conventions (path under `.claude/worktrees/`, branch `worktree-<name>`, base `origin/HEAD`) plus lifecycle. Having done it by hand once is what saves you when something goes sideways.
- **One worktree per ticket, same name as the session.** `claude -w ORD-2231 -n ORD-2231`. The ticket ID already named the branch, the PR and the session; the worktree wasn't going to be the exception.
- **`.worktreeinclude` committed to the repo.** The list of "what the service needs that git ignores" is team knowledge, not personal folklore.
- **Decide the runtime owner up front.** It avoids the "who killed my container?" debugging session — which always arrives at the worst possible moment of both tasks.

## Real limitations

- **Disk and duplicated builds.** The `.git` is shared; `vendor/`, `node_modules/` and build caches are not. Each worktree compiles from cold the first time. In Go the global module cache softens it; in Node it hurts in full.
- **One branch, one worktree.** Git refuses to check out a branch that's already checked out in another worktree: `fatal: 'feature/ORD-2210' is already used by worktree...`. Reasonable once you know what a worktree is; cryptic when you don't. It's usually the first symptom of a forgotten worktree.
- **The runtime is still singular.** Worktrees isolate files, not ports or databases. Gotcha 2 is mitigation, not a solution.
- **Auto-cleanup has fine print.** It applies to interactive sessions; with `claude -p` the worktree stays behind. And manual ones don't collect themselves: `git worktree list` every so often, `git worktree remove <path>` when you're done.

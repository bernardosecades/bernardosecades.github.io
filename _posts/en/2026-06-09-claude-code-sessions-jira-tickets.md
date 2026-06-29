---
title: "Claude Code: three tasks in flight, zero context lost — name sessions after the ticket"
date: 2026-06-09 00:00:00
lang: en
ref: claude-code-sessions-jira-tickets
tags: [claude-code, sessions, workflow, jira]
read_min: 4
excerpt_text: "Merge conflicts land three days after you moved on to another ticket. The session that knew every decision behind that PR is buried under auto-generated titles. Name each session after its Jira ticket and claude --resume becomes an index of everything you have in flight."
---

Wednesday: I finish ORD-2143, a retry policy for the orders webhook, open the PR and jump straight into BIL-987 because the sprint doesn't wait. Friday: a teammate merges first and my PR wakes up with conflicts. That same afternoon, QA asks why a feature flag I finished on Monday — merged, on staging, not yet in production — behaves oddly.

Both answers live in Claude Code sessions I closed days ago. And this is what `claude --resume` shows me:

```text
❯ refactor the handler so retries don't dup…   2 days ago · 47 messages
  fix the webhook test that fails on CI        2 days ago · 12 messages
  add a feature flag for the new flow          4 days ago · 31 messages
  refactor the handler to extract the retr…    5 days ago · 58 messages
  investigate duplicate events in staging      6 days ago · 23 messages
```

Five sessions, all plausible, none called ORD-2143. The auto-generated titles describe the first thing I typed, not the task. So I open two of them, confirm they're the wrong ones, give up and rebuild the context by hand: re-read the ticket, the diff, my own PR description. Fifteen minutes to recover what a session already knew.

The fix costs three seconds per task: name the session after the ticket.

## Name the session when you start the task

Two ways, same result:

```bash
# at launch
claude -n ORD-2143
```

```text
# or mid-session, the moment you realize you forgot
/rename ORD-2143
```

The convention: **one session per ticket, and the name is exactly the ticket ID**. The same key that already names the branch (`feature/ORD-2143-webhook-retries`), the PR title and the Jira card now names the session too. One identifier, four systems.

## --resume as a task index

Once sessions carry ticket IDs, `claude --resume` stops being a list and becomes an index:

```bash
claude --resume        # picker: type ORD-2143, one match, Enter
claude -r ORD-2143     # exact name match — resumes directly, no picker
```

The details that make it work:

- The picker search matches session names, conversation summaries and first prompts. Typing a ticket ID is deterministic in a way "webhook" never is — only one session in your history contains `ORD-2143`.
- If the value you pass isn't an exact name match, the picker opens pre-filled with it as the search term. Either way you're one keypress from the right session.
- Each entry shows the git branch, time since last activity and message count — the branch name confirms you're about to resume the right thing.
- `Ctrl+A` widens the search to all projects, `Ctrl+B` filters by the current git branch. And `/resume` works from inside a session too.

New to these commands? The [continue, resume and rewind basics](/claude-code-sessions-rewind/) cover what `--continue`, `--resume` and `/rewind` actually do; this post is the naming convention layered on top.

## Workflow 1: merged but not deployed

Deploys are batched; tasks are "done" days before they reach production. Bug reports and QA questions arrive asynchronously — always about the task you're no longer in.

`claude -r ORD-2143` brings back the session that made the change: the edge cases we discussed, the alternative we discarded, why the handler checks the timestamp before the status. Answering "why does staging behave like this?" takes one question inside the right session, instead of an archaeology pass through the diff.

## Workflow 2: PR conflicts resolved with the original context

A fresh session resolving conflicts has exactly one source of truth: the diff. It re-derives intent from code, and intent is precisely what conflict hunks destroy. The original session doesn't have to guess — it knows which side of the conflict is load-bearing, because it wrote it.

So when conflicts land:

```bash
claude -r ORD-2143 --fork-session
```

`--fork-session` copies the conversation into a new session and leaves the original untouched — the same pattern as [forking a session to try an alternative](/claude-code-fork-session/). The ticket session stays clean as the stable thread; the conflict resolution happens in a branch of it. One caveat: permissions approved with "allow for this session" don't carry over to the fork.

A related shortcut: if Claude created the PR from inside that session, `claude --from-pr 4812` (PR number or URL) resumes the linked session directly — the ticket naming covers everything that never reached the PR stage.

## Impact

- Recovering a task's context went from ~15 minutes of re-reading ticket, diff and PR description to under a minute: type the ticket ID, Enter, ask.
- Conflict resolutions stay consistent with the original intent — the session that wrote the code resolves its own conflicts.
- It scales with parallel work: three tickets in flight or eight, the index works the same.
- Cost: three seconds per task, zero infrastructure. No plugin, no hook, no script — a naming convention.

## Technical decisions

- **The ticket ID is the whole name.** Not `ORD-2143-webhook-retries` — the picker already shows the branch and the conversation summary next to it; a suffix adds typing without adding precision.
- **One session per ticket.** The docs don't define what an exact-name resume does when two sessions share a name, so I don't create that situation. If a ticket needs a second thread, I fork — the fork gets its own ID and stays grouped under the original in the picker.
- **Name at launch, not at the end.** `claude -n ORD-2143` is muscle memory right after opening the ticket. Renaming at the end depends on remembering — and the sessions you forget are exactly the ones you'll need.
- **Raise `cleanupPeriodDays`.** Sessions are deleted at startup after 30 days by default. Tasks waiting on a deploy can outlive that. In `~/.claude/settings.json`: `{"cleanupPeriodDays": 90}`.

## Real limitations

- **It's discipline, not automation.** Nothing enforces the convention — there's no built-in way to derive the session name from the branch. It survives because it's cheap, not because it's guaranteed.
- **Sessions are local to the machine.** `~/.claude/projects/` doesn't sync. Switch laptops and the index stays behind.
- **Finding the session ≠ the session remembers everything.** If the conversation was long enough to hit compaction, the early context comes back [summarized, not verbatim](/claude-code-context-window/). The convention finds the right session; it doesn't make it infinite.
- **Duplicate names are undefined behavior.** Nothing stops two sessions from sharing a name, and what `claude -r <name>` does then isn't documented. The one-session-per-ticket rule exists so I never find out.

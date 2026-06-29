---
title: "Claude Code: stop watching the terminal — run long commands in the background"
date: 2026-07-21 00:00:00
lang: en
ref: claude-code-background-tasks
tags: [claude-code, background, workflow, productivity]
read_min: 4
excerpt_text: "An integration suite that takes four minutes turns into four minutes of you staring at a frozen terminal, blocked on a result you can't act on yet. Run it in the background instead: Claude fires the command, keeps working on something else, and re-activates itself the moment it exits — with the result already in hand."
---

A long command is a small tax you pay over and over. `make test-integration` takes four minutes; a Docker image build, longer. You hit Enter and then you wait — the terminal is busy, Claude is busy, and you sit there watching a progress bar finish a job you can't act on until it's done. Multiply that by every test run in a day and the waiting adds up to real time, spent doing nothing.

The fix is to stop treating "run a command" as something that has to block. Run it in the background.

## The idea: launch it in the background and keep going

Instead of letting a long command hold the session hostage, you ask Claude to run it in the background. The command detaches and keeps running across turns, and Claude doesn't sit and wait for it — it moves straight on to the next thing: editing another file, drafting the next change, answering a question. You're both working while the suite churns.

The part that makes this more than a convenience: **when the command exits, Claude is re-invoked automatically with the result.** It's not fire-and-forget where you have to remember to come back and check. The moment the tests finish — pass or fail — Claude wakes up, reads the output, and folds it into whatever comes next. If they failed, it starts on the fix; if they passed, it moves to the next step. You never had to babysit the run.

So the loop changes shape. Without background tasks it's *run → wait → react*. With them it's *run → keep working → get pulled back when it matters*.

## How it looks in practice

```text
> run the integration suite, then start wiring up the new endpoint

I'll kick off the suite in the background and start on the endpoint
while it runs.

  ⏵ make test-integration  (background)

[Claude edits handler.go, adds the route, writes the request struct…]

  ✓ background task finished: make test-integration — exit 0
    142 passed, 0 failed (3m51s)

Tests are green. The endpoint changes are in too — here's what I added…
```

While the suite runs you can keep steering. `/tasks` shows what's still going:

```text
> /tasks

  ⏵ make test-integration   running   2m10s
```

And you don't have to wait for the end to peek — you can ask Claude to check the output of a task that's still in flight if you suspect it's hanging on one slow test. The point is the run is *visible and addressable*, not a black box you're locked out of until it returns.

## Background ≠ scheduled

Worth drawing a line here, because it's easy to conflate the two. A background task lives and dies with your session: it's a one-off long job you want to overlap with other work *right now*. It is not a recurring or durable thing. If what you want is "run this every 15 minutes" or "kick this off at 9am tomorrow," that's a scheduled task (`/loop`, `/schedule`) — a different tool with different lifetime rules. Background is for *this* test run, *this* build, while you get on with the rest of the change.

## Impact

- **You chain work instead of waiting on it.** The minutes a long command used to cost are now spent on the next edit, not on a progress bar.
- **The terminal stops being a bottleneck.** One slow suite no longer serializes everything behind it — the run and the work happen side by side.
- **No babysitting.** Because Claude is pulled back automatically on exit, you don't have to remember to return and check. The result finds you.

## Decisions

- **Drive it through Claude, not fire-and-forget.** The reason to ask Claude to run the command in the background — rather than just detaching it yourself — is the automatic re-invocation on exit. That's what turns "a job running somewhere" into "a result that comes back and gets acted on."
- **Background for long and one-off; foreground for fast.** A four-minute suite or a build is the sweet spot. A command that returns in a second gains nothing from backgrounding — the overhead isn't worth it.
- **Keep it visible.** `/tasks` and the ability to inspect a running task mean you're never guessing whether something is still alive. Use them when a run feels stuck.

## Limitations

- **It doesn't survive the session.** Background tasks are scoped to the live session — they're not restored by `claude --resume` or `--continue`. If you close the session, the run goes with it.
- **The result still costs context.** When Claude is re-invoked with the output, that output enters the context window like any other tool result. A command that prints thousands of lines on completion isn't free just because it ran in the background.
- **Not for recurring work.** If you find yourself wanting the same command on a timer, you've outgrown background tasks — reach for `/loop` or a scheduled agent instead.
- **Don't background trivial commands.** For anything that returns near-instantly, plain foreground is simpler and clearer. Reserve this for the runs long enough that the wait actually hurts.

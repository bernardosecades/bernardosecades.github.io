---
title: "Claude Code: /goal — set the finish line, not each step"
date: 2026-05-21 00:00:00
lang: en
ref: claude-code-goal
tags: [claude-code, goal, automation, workflow]
read_min: 4
excerpt_text: "Describe the end state once and walk away. /goal turns prompt-after-prompt iteration into a single condition Claude works toward until it's met."
unlisted: true
sitemap: false
---

The [previous post on subagents](/claude-code-subagents/) was about offloading exploration to a separate context. This one is about something different: how to stop typing *"keep going"*, *"now fix the next one"*, *"run the tests again"* after every turn.

By default Claude finishes a turn and stops. If your task needs twenty iterations, that's twenty messages from you. `/goal` changes the contract: you describe the end state once, and Claude iterates on its own until it gets there.

## What `/goal` does (this is the key)

You type `/goal <condition>` and the first turn starts. After each turn, a small model (Haiku by default) reads the output and judges whether the condition was met.

- If **not**, Claude starts the next turn automatically, with the evaluator's reason as guidance (*"not met — TestParseDate still red"*).
- If **yes**, the goal is cleared and Claude stops.

In the statusline you see `◎ /goal active` with elapsed time. Useful for remembering it's running when you come back from coffee.

## The evaluator

```text
You ──── /goal "go test ./... exits 0"

  turn 1 ─▶ Claude edits foo.go, runs tests, 3 fail
            evaluator: "not met — 3 failures in foo_test.go"
  turn 2 ─▶ Claude fixes assertions, runs tests, 1 fails
            evaluator: "not met — TestParseDate still red"
  turn 3 ─▶ Claude fixes the parser, runs tests, all pass
            evaluator: "met ✓"
            /goal cleared
```

Important detail: **the evaluator only sees what Claude shows in the transcript**. If Claude doesn't print the `go test` output, the evaluator has nothing to judge. This changes how you write the prompt — more on that below.

## `/goal` is not `/loop`, nor a regular prompt

The confusion is constant:

| Command | The next turn starts when… | It stops when… |
|---|---|---|
| Regular prompt | You launch it | You decide |
| `/goal <cond>` | The previous turn ends | The condition is met |
| `/loop 5m <cmd>` | The interval elapses (every 5 min) | You stop it |

Summary: **`/goal` is a condition, `/loop` is a clock**. If you want Claude to stop when something is met, use `/goal`. If you want it to repeat a task every X minutes no matter what, use `/loop`.

## Example 1: the test loop

The simplest case, any dev gets it:

```text
/goal go test ./... exits 0, don't touch anything in test/fixtures/
```

Claude edits, runs tests, fails, reads the output, edits again, runs, fails, edits, passes. You're on Slack meanwhile.

The condition works because it has a **check verifiable in the output**: `go test ./...` exits with a code, the evaluator sees it. *"Make the tests pass"* would also work. *"Improve code quality"* would not — there's nothing the evaluator can measure.

## Example 2: a cascading refactor

You have a shared lib `auditlib` used by six Go microservices: `orders`, `billing`, `notifier`, `gateway`, `accounts`, `reports`. You want to add an options parameter to its main function and keep the build green across all six.

```text
/goal change the signature of auditlib.Log(ctx, event) to
auditlib.Log(ctx, event, opts LogOptions), update the callers
in orders, billing, notifier, gateway, accounts and reports, and
make `go build ./...` pass in the six repos. Stop after 15 turns.
```

What happens, turn by turn:

- Claude edits `auditlib/log.go` and bumps the version.
- Repo by repo: `cd ~/projects/orders && go get auditlib@latest && go build ./...`.
- In `gateway` it trips on a typed wrapper that wraps `auditlib.Log` — the build fails and the evaluator returns *"not met — gateway doesn't compile"*. Claude fixes the wrapper and moves on.
- When the six repos build, the evaluator gives the ok.

Watch the `stop after 15 turns`: it's the emergency brake. Without it, an impossible condition (a repo that never compiles for an unrelated reason) will burn tokens until you kill it yourself.

## The condition is what matters

Common mistakes at the start:

- **Vague:** *"clean up the code in module X"* — the evaluator can't judge. Don't use it.
- **Not verifiable in the transcript:** *"the database is optimized"* — doesn't show up in the output. Don't use it.
- **No turn limit:** impossible conditions burn tokens until you stop them. Always add `stop after N turns`.
- **Confusing it with `/loop`:** you saw it above, but it happens all the time.

Reliable pattern: *"&lt;concrete command&gt; exits with &lt;code / count / state&gt;, stop after N turns"*.

## Why this matters

`/goal` doesn't make Claude smarter. It frees it from asking your permission after every turn.

You use it when **the goal is clear** but **the path isn't**: red tests, broken builds, cascading refactors, migration queues, lint warnings that need to go to zero.

If the goal can't be measured in the output, don't use `/goal`. Use `/plan` and steer it turn by turn.

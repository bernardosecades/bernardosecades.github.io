---
title: "Claude Code: subagents have their own context window"
date: 2026-05-19 00:00:00
lang: en
ref: claude-code-subagents
tags: [claude-code, subagents, context, workflow]
read_min: 4
excerpt_text: "A subagent isn't a helper that shares your session — it's a Claude with its own context window. Use it to soak up the exploration cost so your main session keeps the summary, not the noise."
video:
  src: /assets/videos/demo_subagents.webm
  thumbnail: /assets/videos/demo_subagents.jpg
  duration: PT2M5S
  width: 1917
  height: 1038
---

The [previous post on the context window](/claude-code-context-window/) name-dropped subagents as one of the four habits: *"it has its own window, it returns a summary"*. Most people I talk to nod and keep treating subagents as a second tab on the same conversation — same memory, same files, same notes. They are not.

A subagent is a separate Claude invocation, with its own context, that runs the prompt you hand it and returns **one final text message**. Nothing else crosses the boundary.

## Its own context window (this is the key)

The subagent does not see your conversation. It does not see the files you injected with `@`. It does not see the decisions you discussed in chat. Only the prompt you pass it.

And on the way back, the main session receives **a single chunk of text** — the subagent's final assistant message. No tool-call transcript, no file contents, no intermediate thinking.

```text
Main session                Subagent (Explore)
┌──────────────┐            ┌──────────────────────┐
│ context: 40k │            │ context: 0 → 80k     │
│              │  prompt    │                      │
│              │ ─────────▶ │ greps, reads, finds, │
│              │            │ reads more files…    │
│              │ ◀───────── │                      │
│ context: 41k │  summary   │ context: 80k (gone)  │
└──────────────┘  (~1k)     └──────────────────────┘
                                discarded on exit
```

Main pays for the prompt (~1k) and the summary (~1k). The 80k spent grepping lives and dies in the subagent.

## Built-in vs custom

`Explore` and `general-purpose` ship with Claude Code. `Explore` is read-only and tuned for codebase navigation — the one I reach for 90% of the time.

A **custom** subagent is a markdown file in `.claude/agents/<name>.md` with frontmatter and a body that is its system prompt:

```yaml
---
name: auditlib-callsites
description: Finds call sites of an auditlib function across the
  microservices. Returns repo / file:line / snippet.
tools: [Bash, Read, Grep, Glob]
model: claude-sonnet-4-6
---
```

Worth defining one when you'd type the same orchestration prompt every week, when you want to restrict tools to read-only, or when you want a cheaper model for grunt work. Not worth it for a one-off.

A custom subagent in `.claude/agents/` is **not** the same as a skill in `.claude/skills/`. A skill is a reusable instruction bundle the main session loads. A subagent is a separate Claude with its own window. A skill can launch a subagent; they are not the same thing.

## How they get invoked

Three modes:

- **Auto** — mid-task, the main model decides on its own: *"I'll launch an Explore subagent to map this."* Visible in the transcript, interruptible.
- **Explicit** — you ask for it: *"use the `auditlib-callsites` subagent to find all callers of X."* Claude picks the right one when an agent's `description` matches.
- **Nested** — a custom orchestrator can spawn its own subagents in parallel, useful when you'd otherwise serialize a fan-out.

The contract is always the same: prompt in, one final text message out. That's the whole interface.

<video controls playsinline preload="none" poster="/assets/videos/demo_subagents.jpg">
  <source src="/assets/videos/demo_subagents.webm" type="video/webm">
</video>

## Two real examples from the workspace

I work across ~25 Go microservices in `~/projects/` (the [multi-repo workspace setup](/claude-code-multi-repo-workspace/)). Two daily uses.

**Who calls `auditlib.Submit()`?** I type at the main:

> Launch an Explore subagent on `~/projects/`. Find every call site of `auditlib.Submit` across the microservices (orders, billing, notifier, gateway, accounts, reports, etc.). For each hit return: repo, file:line, the calling function, and one line of context.

What lands in the main session:

```text
4 call sites:
- orders    internal/order/engine.go:142     func (e *Engine) finalize
- billing   internal/worker/process.go:88    func (w *Worker) onCharge
- notifier  internal/usecase/dispatch.go:201 func (s *Service) send
- gateway   internal/handler/submit.go:55    func (h *Handler) Post

All four pass a *auditlib.Event built locally; none reuse a
shared builder. orders is the only one that wraps the call in a retry.
```

Order of magnitude: ~25k tokens burned grepping and reading inside the subagent; my main grew by ~200. Without it, that 25k would have polluted my context for the rest of the session.

**Impact of changing `httpx.Foo()`'s signature.** I type:

> I want to change `httpx.Foo(ctx, id string)` to `httpx.Foo(ctx, id string, opts FooOptions)`. Use the `Explore` subagent on `~/projects/` to list every consumer grouped by repo. For each, tell me whether the call would compile with `opts = FooOptions{}` as default, or whether the caller exposes the current shape.

What comes back:

```text
9 callers across 6 repos. All direct calls.

Safe with default opts:
- orders, billing, notifier, gateway, accounts (8 sites total)

Watch:
- reports  internal/legacy/foo_wrapper.go:18
  Wraps httpx.Foo behind a typed func ReportFoo, exposed in the
  public API. Changing httpx.Foo cascades into ReportFoo's callers.
```

~40k tokens inside the subagent across 6 repos; my main got 400 tokens of structured verdict — and the cascading-break hint about `reports`. The kind of thing I would not spot grepping by hand at 11pm.

## Why this matters

The subagent is a **context-budget tool**, not a "make Claude smarter" tool. Same model behind it (or a smaller one). What changes is what crosses the boundary back to you.

Rule of thumb: if a sub-task is going to read more than five files or run more than three greps, launch one. Built-in `Explore` covers most of it. Custom agents only when you'd otherwise repeat the same orchestration prompt every week.

The cost of spawning one is small. The cost of letting that exploration smear across the main context lasts the rest of the session.

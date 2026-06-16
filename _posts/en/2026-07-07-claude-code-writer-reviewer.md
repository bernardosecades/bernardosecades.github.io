---
title: "Claude Code: the writer/reviewer pattern — don't let the author review their own diff"
date: 2026-07-07 00:00:00
lang: en
ref: claude-code-writer-reviewer
tags: [claude-code, subagents, code-review, workflow]
read_min: 4
excerpt_text: "Asking the session that wrote the code to review it isn't review — it's confirmation: every shortcut is already justified in its context. A reviewer subagent judges the same diff from a clean window, and suddenly the findings show up."
---

Before pushing a branch, I used to type the responsible-sounding thing: *"review this code before I push."* The review came back in seconds — a couple of style remarks and a thumbs up. Then a teammate would find a duplicated event on the retry path, in the exact function that review had just blessed.

The model didn't fail. The setup did. I asked the author.

## Why self-review fails (this is the key)

The session that wrote the code carries its whole creative history in context: the requirements as it understood them, the alternatives it discarded, the justification for every shortcut. When you ask that same session to review, it doesn't re-derive the conclusions — it re-reads them. The bug isn't invisible because the model is weak; it's invisible because the reasoning that produced it is sitting right there in the window, already accepted.

Human code review solved this ages ago: the author doesn't approve their own PR. The fix was never a smarter author. It was a second pair of eyes with no stake in the outcome.

## The pattern: split the writer from the reviewer

Two roles, two contexts:

- The **writer** is your main session, doing what it already does: implements, runs the tests, gets the branch green.
- The **reviewer** starts blind: a clean context, the diff, the acceptance criteria. No rationale, no chat history, no investment.

In Claude Code the isolation comes free: [subagents have their own context window](/claude-code-subagents/) and they do not see your conversation. In that post the boundary was a *cost* device — exploration noise dies inside the subagent. Here the same boundary is the *correctness* device: the reviewer can't inherit the writer's bias, because the writer's reasoning never crosses.

## The reviewer agent

A markdown file in `.claude/agents/diff-reviewer.md`, committed to the repo:

```markdown
---
name: diff-reviewer
description: Reviews the current branch's diff against main. Read-only.
  Returns findings with severity and file:line, or an explicit approval.
tools: [Bash, Read, Grep, Glob]
---

You are a code reviewer. You did not write this code and you have no
stake in it. Your job is to find problems, not to approve.

1. Run `git diff main...HEAD` yourself. Do not accept a summary of the
   changes from anyone — the diff is the source of truth.
2. Read enough surrounding code to judge each change in context.
3. Report only findings that affect behavior, correctness, or security.
   Style nits: skip them unless the repo's linter would flag them.

For each finding: severity (blocker / should-fix / nit), file:line,
what breaks and under which input. If a real pass turns up nothing,
say "approved" and list what you checked — never pad findings to
look thorough.
```

Two details do the heavy lifting: the tool list has **no Edit or Write**, and step 1 makes the reviewer **pull the diff itself** instead of trusting anyone's summary. Both are deliberate — more in Decisions.

## The loop

Writer implements. Then, from the same session:

```text
> use the diff-reviewer subagent on this branch

3 findings:

1. [blocker] internal/worker/process.go:88 — the retry wraps the
   charge AND the audit emit: on a retried charge the event goes
   out twice. Duplicate events for every downstream consumer.
2. [should-fix] internal/handler/submit.go:61 — the err from
   body.Close() is shadowed by the deferred assignment; a failed
   close is silently dropped.
3. [should-fix] internal/store/list.go:47 — cursor comparison uses
   >= on created_at; when a page boundary lands on equal timestamps,
   the last item repeats on the next page.
```

Finding 1 had been "reviewed" by the writer session minutes earlier — and waved through. Same model, same diff. The only thing that changed is that the reviewer's window doesn't contain the paragraph where the writer convinced itself the retry was safe.

The writer fixes all three, then re-dispatches:

```text
> use the diff-reviewer subagent on this branch again

approved — checked the retry path (emit now outside the retry
closure), deferred Close error handling, and cursor boundaries
with equal timestamps. No further findings.
```

Order of magnitude: the reviewer burned ~30k tokens pulling the diff and reading around it; my main session received ~600 back. The review cost a window I was never going to keep anyway.

## Impact

- **Findings the writer session had already waved through** — the duplicated event on retry, the swallowed `Close` error. Same model finds them once it stands outside the writer's context.
- **The main context stays clean.** The review's grepping and reading dies in the subagent; the writer gets a findings list, applies fixes, keeps going.
- **The review standard is versioned.** `.claude/agents/diff-reviewer.md` lives in the repo: every teammate — and every session — gets the same reviewer, not whatever prompt someone improvises at 6pm.

## Decisions

- **The reviewer is read-only.** No `Edit`, no `Write`. A reviewer that can patch starts patching, and you lose the boundary that made its judgment trustworthy. Findings cross back as text; the writer applies them where the full context lives.
- **The reviewer runs `git diff` itself.** The dispatch prompt is written by the writer session — if it carries "I added a retry, which is safe because…", the bias you just isolated away rides back in. The orchestrator passes the branch and the acceptance criteria, nothing else.
- **Severity ladder in the prompt.** "Find problems" with no bar produces twelve nits per pass and the loop never converges. Blocker / should-fix / nit, plus an explicit "don't pad", keeps passes short and honest.
- **Two rounds, then a human.** Write → review → fix → review. Whatever survives the second pass goes to a person, not a third pass — repeated rounds converge on the reviewer's taste, not on correctness.

## Limitations

- **Context independence, not model independence.** Both roles run on the same weights; blind spots of the model survive the split. Setting a different `model:` in the reviewer's frontmatter diversifies a little — a human review for risky changes is still the backstop.
- **It's static analysis by default.** The reviewer reads the diff; it doesn't run the suite unless you tell it to. You can allow `make test` in its prompt, at the price of slower passes.
- **A full agent pass per round.** Over a large diff that's real tokens and real minutes. For a three-line fix, read the diff yourself.
- **The dispatch can still leak framing.** "Branch name + criteria only" is a discipline, not a mechanism. If the writer session editorializes in the dispatch prompt, some bias travels — the `git diff` rule shrinks the channel, it doesn't close it.

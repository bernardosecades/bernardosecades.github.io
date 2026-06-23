---
title: "Claude Code: let it grill you on the plan before you write any code"
date: 2026-07-14 00:00:00
lang: en
ref: claude-code-grill-me
tags: [claude-code, plan-mode, skills, workflow]
read_min: 4
excerpt_text: "A plan reads clean right up until you start implementing it — then the decisions nobody resolved surface one by one, at the worst possible time to change them. The grill-me skill drags them out first: it interviews you relentlessly, one question at a time, recommending an answer for each, and explores the code instead of asking when it can answer itself."
---

A plan reads clean. The steps are in order, the files are named, it looks ready. Then I start implementing and the questions begin: what happens on this edge case? This dependency or that one? This format or the other? None of them were in the plan — not because the plan was sloppy, but because writing prose doesn't force you to resolve every branch. You discover the unresolved ones mid-code, which is exactly when changing your mind is most expensive.

The skill I reach for to stop that happening is `grill-me`, and it pairs almost perfectly with plan mode.

## The idea: get interrogated before you write code

`grill-me` is a tiny skill whose whole job is to turn Claude into a relentless interviewer about a plan or design. It walks down the decision tree branch by branch, resolving dependencies between decisions one at a time, and — this is the part that makes it usable — **it recommends an answer to every question it asks.** So you're not staring at an open-ended "what do you want?"; you're reacting to a concrete proposal, which is much faster.

Two rules in its instructions make the difference:

- **One question at a time.** No wall of fifteen bullet points to triage. Each answer can change what it asks next, so the questions stay relevant and the dependencies get sequenced for you.
- **Explore the code instead of asking, when it can.** If a question is answerable by reading the repo, it goes and reads the repo rather than making you the lookup service.

That's the whole skill. No script, no magic — just a sharp prompt.

## Why it fits plan mode

Plan mode already puts Claude in read-only, exploration mode: it can read files and search the codebase but can't edit anything until you approve. That's the exact posture `grill-me` wants. Instead of Claude quietly drafting a plan and handing it to you to rubber-stamp, it stress-tests the design *with* you — and when a question can be settled by grepping the code, it just does that, because it already has read access and nothing to write.

The result is a plan that arrives with its branches already resolved, not one that looks done and unravels on the first commit.

## How to install it

You need Claude Code with plan mode (cycle modes with **Shift+Tab**). Then there are two ways to get the skill.

### The quick way: the installer

`grill-me` started life in Matt Pocock's skills collection — it's the one that [went viral](https://www.aihero.dev/my-grill-me-skill-has-gone-viral). The fastest way to get the original is his community installer:

```bash
npx skills@latest add mattpocock/skills
```

It opens a picker: choose `grill-me` (and its sibling `grill-with-docs` if you want it) and which agent to install it on. The skill lands in `~/.claude/skills/`. Note this pulls the full, original version of the skill, which is richer than the stripped-down one below. The source lives at [github.com/mattpocock/skills](https://github.com/mattpocock/skills).

### Or build it yourself

If you'd rather have a single self-contained file with no dependencies — or just want to see how little it takes — it's a personal skill you create once and use in every project:

```bash
mkdir -p ~/.claude/skills/grill-me
```

`~/.claude/skills/grill-me/SKILL.md`:

```markdown
---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.
```

That minimal version is my own stripped-down take — enough to reproduce the behaviour in this post. Drop it under a project's `.claude/skills/` instead if you'd rather scope it to one repo. To fire it either way: enter plan mode, sketch what you want to build, and say **"grill me."** The `description` is what lets Claude pick it up from that phrase.

## In practice

A real session looks like this — plan first, then hand the wheel over:

```text
> plan: add automatic retries for failed outbound notifications
> grill me

Q1 — Where should the retry state live? In-memory, or persisted in a
`notification_attempts` table?
Recommended: persist. An in-memory queue loses everything on a deploy,
and a dropped notification is exactly what you can't afford here.

> agreed, persist it

Q2 — How many attempts before giving up, and with what backoff?
Recommended: 5 attempts, exponential backoff capped at 1h — enough to
ride out a downstream blip without hammering a service that's down for
the day.

> make it 6, cap at 30m

Q3 — Do retries need to preserve per-recipient order, or is each
notification independent?
(checking the code…) The notifier already sends each notification on
its own goroutine with no ordering guarantee, so retries add no new
constraint — I'll treat each attempt as independent.

> correct, independent
```

Three questions, three branches closed — and Q3 it answered by reading the code, not by making me recite how the sender works. By the time the plan is written, the table, the backoff policy, and the ordering assumption are all decided, on the record, before a single line exists.

## Impact

- **Implicit decisions surface before implementation, not during.** The edge cases I used to discover mid-code show up as questions while changing the answer still costs nothing.
- **Less rework.** The plan that gets approved already has its branches resolved, so the first commit doesn't immediately contradict it.
- **A faster interview than open-ended planning.** Reacting to a recommended answer is quicker than authoring one from scratch — I mostly say "yes" and occasionally "no, do X."

## Decisions

- **One question at a time, on purpose.** A batch of fifteen questions is a triage chore; a sequence lets each answer steer the next and forces the dependencies into order.
- **It recommends, I decide.** The recommended answer means I can move at the speed of agreement and only spend effort where I disagree — without giving up the final call.
- **Explore over ask.** Anything the codebase can answer, it answers from the codebase. Plan mode's read access makes that free, and it keeps the questions to the things only I know.
- **Grill in plan mode, not after.** The cheapest moment to resolve a branch is before any code commits to it.

## Limitations

- **It stress-tests a plan; it doesn't invent one.** You still need a rough idea of what you're building — `grill-me` sharpens direction, it doesn't supply it.
- **Quality scales with context.** The more relevant code there is to explore, the better the questions. On a greenfield repo with little to read, it leans more on you.
- **It lengthens planning.** That's the trade: more minutes up front to buy back the mid-implementation rewrites. For a one-line fix it's overkill.
- **No code comes out.** It's a design conversation. The output is a resolved plan — you (or the next phase) still have to build it.

---
title: "Your AI plan dies with the session. Put the spec in the repo"
date: 2026-10-13 00:00:00
lang: en
ref: openspec-spec-in-repo
tags: [openspec, spec-driven, workflow, claude-code]
read_min: 5
excerpt_text: "You spend half an hour agreeing a plan with Claude, close the session, and the plan is gone. The next session re-derives it slightly differently, and the reviewer checks the PR against your memory. OpenSpec moves the agreement into the repo, where it gets versioned, reviewed, and used as the contract."
---

You spend half an hour agreeing a plan with Claude: the edge cases, the state transitions, what each service does. Then the session ends and the plan goes with it. The next session works it out again, a little differently. The reviewer checks the PR against what you *remember* agreeing. On a change that spans three repos, "a little differently" is how bugs reach production.

The fix isn't a better plan. It's putting the plan somewhere that outlives the session. I've been using [OpenSpec](https://github.com/Fission-AI/OpenSpec) for that, with Claude Code doing the work.

## What OpenSpec adds

OpenSpec is a lightweight spec layer that lives in your repo. `openspec init` creates a folder and installs a few `/opsx:*` slash commands in Claude Code:

```
openspec/
├── specs/                  # source of truth: how the system behaves today
└── changes/
    └── add-manual-review/
        ├── proposal.md     # why, and what changes
        ├── design.md       # technical approach
        ├── tasks.md        # implementation checklist
        └── specs/          # deltas: ADDED / MODIFIED / REMOVED requirements
```

Each requirement in a delta comes with scenarios (*WHEN this happens, THEN that*). When the change ships, `/opsx:archive` merges the deltas into `specs/`, so the source of truth stays current. That's all there is to it: Markdown files, in git, next to the code they describe.

## The change

The real case: add a **manual-review state** to an onboarding flow. `accounts` owns the flow. `billing` has to hold activation while an account is under review, and the client SDK has to expose the new state. That's two services and one SDK. Without a spec, a change this size is where I'd re-explain the feature to Claude three times and still find a mismatch in review.

## My flow, step by step

**1. Explore, then grill.** `/opsx:explore` to shape the idea, with [grill-me](/claude-code-grill-me-plan-mode/) on top so Claude interviews me one decision at a time instead of drafting something for me to rubber-stamp. This is where the edge cases came out: what happens if the account is *already* in review, who can release it, what `billing` does with a charge that's in flight.

**2. Propose.** `/opsx:propose add-manual-review` writes the decisions down as `proposal.md`, `design.md`, `tasks.md` and the spec delta. The grill-me answers stop being chat history and become files.

**3. Review the artifacts before any code.** I read the delta's scenarios the way I'd read a PR. It's the cheapest point to catch a misunderstanding: fixing a line of Markdown beats rewriting a handler. A couple of scenarios got rewritten here.

**4. Apply in a clean context.** `/opsx:apply` in a fresh session. It needs nothing from the session before it, because the files hold the whole agreement. This is where the thesis pays off: the change ran across more than one session, and I picked it up each time with "read `openspec/changes/add-manual-review`", with no re-explaining.

**5. Review against the spec, not against memory.** Before archiving, my [reviewer agents](/claude-code-writer-reviewer/) get the diff *and* the delta. Their question is no longer "does this look right?" but "does every scenario have code that satisfies it, and is there code no scenario asked for?" That's a far easier question to answer well.

**6. Archive.** `/opsx:archive` folds the delta into `openspec/specs/`. The next change to this flow starts from a description of how it actually behaves.

## Multi-repo: where does the spec live?

OpenSpec is per repo and my change wasn't. I put the spec **only in the repo that owns the behavior**, `accounts/openspec/`. `billing` and the SDK change according to `tasks.md`, with no `openspec/` of their own.

The alternatives didn't hold up. One `openspec/` per repo triples the upkeep and leaves three partial truths. A spec at workspace level, in the parent folder of my [multi-repo workspace](/claude-code-multi-repo-workspace/), is tempting because Claude already sees everything from there. But that folder isn't a repo, so the spec isn't versioned or reviewed. That throws away the whole point.

## "Isn't this just plan mode with more steps?"

No, though they work well together. Plan mode settles **how** to implement something in *this* session, and it's gone when the session ends. OpenSpec captures **what** the system must do, in files that get versioned, reviewed in the PR, and carried forward into `specs/`. I still use plan mode inside `apply` for the how. OpenSpec is what keeps the what from drifting.

## Impact

- **Edge cases surfaced before code.** Three decisions that would have shown up as review comments (or incidents) were settled in explore and proposal review.
- **No re-explaining between sessions.** The change ran across several sessions, and each one started from the change folder instead of from my memory.
- **Review with a reference.** Reviewer agents checked the diff against concrete scenarios, which produced specific findings ("scenario X has no code") instead of taste-level opinions.

## Decisions

- **OpenSpec for the what, plan mode for the how.** They complement each other. Neither replaces the other.
- **One spec, in the owning service.** A single versioned source of truth beats per-repo copies or an unversioned workspace folder.
- **Human review of the artifacts before `apply`.** The proposal is the cheapest place to fix a misunderstanding, so it gets a real read, not a skim.
- **Reviewers get the delta, not just the diff.** That turns review into a checklist against scenarios.

## Limitations

- **The spec drifts if nobody updates it.** In this very change, a decision (the manual-review timeout) changed during `apply`, and `design.md` and the delta weren't updated. The reviewer caught it by comparing diff against delta before archive. Now that check is mandatory, because a stale spec is worse than none: it lies with authority.
- **Overkill for small changes.** A proposal, design, tasks and a delta for a 10-line fix is bureaucracy. I reserve OpenSpec for changes that cross services or change behavior other teams rely on.
- **Consumer repos don't see the spec.** The `billing` and SDK PRs don't carry the `accounts` delta, so their reviewers work without it unless I link it by hand.

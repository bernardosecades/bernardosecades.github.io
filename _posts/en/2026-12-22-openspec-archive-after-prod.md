---
title: "A change isn't done when the code merges. We archive it after production"
date: 2026-12-22 00:00:00
lang: en
ref: openspec-change-lifecycle
tags: [openspec, spec-driven, workflow, claude-code, team]
read_min: 5
excerpt_text: "OpenSpec's archive step moves a change's scenarios into the specs that describe the system. If you run it when the code merges, those specs describe develop, and whoever reads them assumes production. This is the full lifecycle of a change in our team, from the first command to the archive after the deploy, and the mistake that made us put the archive last."
---

OpenSpec has a step that turns a change into documentation: `openspec archive` moves the change's scenarios into `openspec/specs/`, the folder that describes how the system behaves. The obvious moment to run it is when the code merges. The problem is that merged code is on develop, and anyone who reads `specs/` will assume they're reading what runs in production. In our team the archive is the last step, after the production deploy, and `tasks.md` says so.

This closes the series on how we use OpenSpec as a team. The earlier posts covered [the central specs repo](/openspec-one-repo-for-all-specs/), [the rules we give the AI](/openspec-config-rules-for-ai-specs/) and [a spec written before a refactor](/openspec-spec-before-refactor/). This one puts all the steps in order.

## The lifecycle

Everything below runs from the workspace parent folder, where the specs repo and the services are cloned side by side.

1. Create the change. `openspec new change unify-delegations` creates the folder in `openspec/changes/`.
2. Write the artifacts. `/opsx:propose` drafts `proposal.md`, `design.md`, `tasks.md` and the spec delta, following the config rules. I usually run [grill-me](/claude-code-grill-me-plan-mode/) first so the edge cases come from questions and not from the draft.
3. Validate. `openspec validate unify-delegations` checks the structure: requirements with SHALL or MUST, scenarios with the right headers.
4. Open the spec PR. It goes to the specs repo and works as a gate. QA and the rest of the team review the scenarios, and no code is written until it's approved.
5. Implement. `/opsx:apply` reads the change and edits the services. Each service gets its own PR, and each finished task gets its `[x]` in `tasks.md`.
6. QA in dev, then deploy to production.
7. Archive. `openspec archive unify-delegations` merges the delta into `specs/` and moves the change to `changes/archive/` with the date in front.

## tasks.md covers delivery too

Our first change had a `tasks.md` with one line, the ticket number. The second one has three sections. This is how it looked while the service PR was in review:

```markdown
## 1. Implementation (PROJ-787)
- [x] 1.1 accounts: integration tests for the tree against a real database,
      green on develop except the two fixes (PR #33)
- [x] 1.2 accounts: single one-level read, tree traversal in the service layer (PR #33)

## 2. Delivery
- [ ] 2.1 accounts: review and merge PR #33 to develop
- [ ] 2.2 QA in dev: GetAccount with `with_delegations`, and ListManagers
- [ ] 2.3 Deploy accounts to production

## 3. Closing
- [ ] 3.1 `openspec archive unify-delegations`
```

Anyone who opens the change knows where it stands without asking. Code done, waiting for review, archive pending.

## What archive does

When every task is ticked, the archive looks like this:

```
Specs to update:
  provider-retry-webhook: create
✔ Proceed with spec updates? Yes
Applying changes to openspec/specs/provider-retry-webhook/spec.md:
  + 1 added
Totals: + 1, ~ 0, - 0, → 0
Specs updated successfully.
Change 'provider-retry-webhook' archived as '2026-10-22-provider-retry-webhook'.
```

If tasks are still open, it asks first:

```
Task status: 0/1 tasks
? Warning: 1 incomplete task(s) found. Continue? (y/N)
```

That warning is the only check, and it takes a single "y" to get past it. Our first change was archived exactly like that, with its only task unticked. The archive went through and `specs/` was updated anyway. With the delivery and closing sections in `tasks.md`, an open task now means something concrete (QA not done, deploy not done), so a "y" there is a decision someone makes knowingly.

## Impact

- **`specs/` describes production.** A change only reaches `specs/` after the deploy, so what's there is what's running.
- **Anyone can see where a change stands.** `tasks.md` shows code, review, QA, deploy and archive, with a PR or ticket next to each one.
- **One sequence for the whole team.** The same seven steps for every change, from the same folder, with the same commands.

## Decisions

- **Archive after production instead of after merge.** OpenSpec also has `/opsx:sync`, which updates `specs/` without archiving the change. We don't use it at merge time for the same reason: it would describe develop.
- **The archive is a task in `tasks.md`.** If it's on the list, it doesn't depend on someone remembering it after the deploy.
- **Delivery steps in `tasks.md`, not only code.** QA and deploy are part of the change. Without them an open task says nothing about where the change is, which is how the first change got archived with its only task unticked.

## Limitations

- **Archiving is manual.** Nothing connects the production deploy with the specs repo. If nobody runs the archive, the change stays in `changes/` and `specs/` falls behind production.
- **Ticking tasks is manual too.** `/opsx:apply` ticks implementation tasks, but QA and deploy are ticked by hand, and the warning on archive is easy to skip.
- **The process is young.** Two changes have gone through it. Things will change with the next ones, and some of what I wrote in this series will probably need a follow-up.

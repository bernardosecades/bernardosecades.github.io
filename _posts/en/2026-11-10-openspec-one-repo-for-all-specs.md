---
title: "The spec lived in one repo and the change touched three. So we gave specs their own repo"
date: 2026-11-10 00:00:00
lang: en
ref: openspec-central-repo
tags: [openspec, spec-driven, workflow, claude-code, team]
read_min: 5
excerpt_text: "A month ago I argued the spec belongs in the repo of the service that owns the behaviour. Then my team adopted OpenSpec, most changes touched two or three services, and reviewers in the other repos never saw the scenarios. We moved every spec into a repo of its own. This is why, how we work with it, and what it costs."
---

A month ago I wrote that the spec should live in the repo of the service that owns the behaviour. Under Limitations I added that the consumer repos wouldn't see it. That limitation turned out to be the main problem. When my team started using OpenSpec together, most changes touched two or three services, and picking one "owner" for the spec was a guess. The PRs in the other repos were reviewed by people who had never read the scenarios.

So we did what I had argued against. We created a repo that holds only specs. Here's why, how we work with it, and what it costs.

## What changed in my thinking

In the [first post](/openspec-spec-in-the-repo/) I rejected a workspace-level spec because the workspace folder isn't a repo, so nothing in it gets versioned or reviewed. I still think that about the folder. What I missed is that the shared place can be a repo of its own. Then the spec is versioned and reviewed like any other code, and no service has to pretend it owns a change that spans three.

## Why a central repo

There were four reasons, and they showed up roughly in this order.

Changes cross services. When `accounts` changes what it returns for a field, `billing` and `reports` receive something different. With one spec per owner, the `billing` reviewer worked from the Jira ticket. Now every service PR links to the same change folder.

People who don't read Go can review. QA and product open the spec PR, read the scenarios and leave comments. When the spec lived inside a service repo, next to the code, nobody outside the developers opened it.

The rules are the same for everyone. `openspec/config.yaml` holds the context and the rules for each artifact, and the `/opsx:*` commands and skills are committed in `.claude/` in the same repo. Everyone on the team generates specs with the same instructions. With one setup per repo, they had started to drift. The config gets its own post next.

There is a catalogue of the system. `openspec/specs/` describes how each capability behaves today, whichever service implements it. When someone joins, or when I ask Claude how a flow works, that folder is the first thing to read.

This is the layout:

```
specs/                          # its own git repo
├── .claude/                    # /opsx commands and skills, shared by the team
└── openspec/
    ├── config.yaml             # context and rules per artifact
    ├── specs/                  # how the system behaves today
    └── changes/
        ├── unify-delegations/
        │   ├── proposal.md
        │   ├── design.md
        │   ├── tasks.md
        │   └── specs/delegations/spec.md
        └── archive/
            └── 2026-10-22-provider-retry-webhook/
```

## How we work with it

The specs repo is cloned next to the service repos, in the same parent folder I already use as a [multi-repo workspace](/claude-code-multi-repo-workspace/). Claude starts in that parent folder. From there `/opsx:propose` writes into `specs/openspec/changes/...`, and `/opsx:apply` reads the change and edits code in `accounts/`, `billing/` or wherever `tasks.md` points.

The spec PR is a gate. It gets approved before anyone writes code. In the first post I said the proposal is the cheapest place to fix a misunderstanding, and with QA reading it too, it's also where a missing scenario shows up before it becomes a bug report. After that, each service gets its own PR, and `tasks.md` lists them all.

## Impact

- **Every reviewer reads the same contract.** Service PRs link to the change folder, so whoever reviews `billing` reads the same scenarios as whoever reviews `accounts`.
- **QA reviews behaviour before implementation.** Comments on scenarios arrive in the spec PR, while changing them still costs a line of Markdown.
- **One setup for the whole team.** One config and one set of `/opsx` commands, versioned together with the specs.
- **Questions about a flow start from `specs/`.** I can find out how something behaves today without reading three services.

## Decisions

- **Central repo instead of a spec per service.** When a change crosses services, a per-repo spec is either duplicated or invisible to half the reviewers.
- **Central repo instead of a workspace folder.** Both give Claude the full picture, but only the repo is versioned and reviewed through PRs.
- **The spec PR is a gate.** No code until the scenarios are approved. Running both PRs in parallel would be faster, but then the spec tends to describe whatever the code already does.
- **Claude runs from the workspace parent.** It's the only place where it sees the change and the services at the same time.

## Limitations

- **Spec and code are in different PRs.** The link between them is manual, a URL in the PR description and in `tasks.md`. If a decision changes during implementation, nothing forces the spec to change with it.
- **Everyone needs the same folder layout.** `/opsx:apply` assumes the specs repo and the services are siblings, so a new teammate has to set that up before the first change.
- **It's early.** Two changes have gone through the process so far, and only one is archived. This is what we've seen up to now, and some of it will probably change.

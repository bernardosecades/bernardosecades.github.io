---
title: "The AI wrote specs that read like code. A config.yaml fixed it"
date: 2026-11-24 00:00:00
lang: en
ref: openspec-config-rules
tags: [openspec, spec-driven, workflow, claude-code]
read_min: 5
excerpt_text: "Our first AI-generated specs named functions, turned a bug we were keeping into a requirement, and had scenarios no test covered. Each teammate also prompted differently, so every change came out with a different shape. OpenSpec's config.yaml gives the AI the same context and rules for every artifact. These are ours and the mistakes behind them."
---

When we started generating specs with `/opsx:propose`, the drafts looked fine at first glance and had problems on a second read. Scenarios named functions and packages. A bug we were keeping on purpose turned into a requirement. Some scenarios had no test anywhere, and nobody noticed until review. On top of that, each of us asked Claude in a slightly different way, so every change came out with a different shape.

OpenSpec has a place for this, `openspec/config.yaml`. In the [previous post](/openspec-one-repo-for-all-specs/) one of the reasons for a central specs repo was having one set of rules for the whole team. This is what those rules look like and where they came from.

## What the config is

`openspec init` creates `config.yaml` with two optional sections, commented out. `context` is free text the AI sees every time it creates an artifact. `rules` holds instructions for each artifact type: proposal, specs, design and tasks. This is ours, translated and trimmed (the real one is in Spanish):

```yaml
schema: spec-driven

context: |
  Go microservices talking over gRPC; proto contracts live in a shared repo.
  Async events over Kafka and RabbitMQ.
  Merchants onboarded on V1 or V2, with different behaviour in some flows.
  Jira: PROJ-* is our team.

rules:
  proposal:
    - Template headers in English (Why, What Changes, Capabilities, Impact), OpenSpec parses them
    - In Impact, list the affected services
    - Say whether a contract changes (proto, topic, queue) and which consumers notice
  specs:
    - 'Each requirement is titled "Requirement: REQ-NN <title>"'
    - Describe the observable contract using proto fields, flags and domain states
    - No function, package or file names (those go in design.md)
    - One scenario per observable rule
    - Don't document defective behaviour that is kept on purpose
  design:
    - Every decision with the alternative we rejected
    - Map each scenario to the test that covers it
  tasks:
    - Jira ticket and PR per repo
    - Include QA in dev and archiving the change after the production deploy
```

## Before and after

Our first change went through with the default, empty config. The proposal came out in English with headers Claude made up (Context, Proposed Changes, Impacted Microservices) instead of the template's. `tasks.md` was a single line with the ticket number. It did the job, but the next change would have looked different, and the one after that too.

The second change was written after we filled in the config. The proposal follows Why, What Changes, Capabilities and Impact, and Impact lists the consumers that will receive different output. Each decision in `design.md` comes with the alternative we discarded. `tasks.md` has three sections: implementation, delivery and closing the change.

## Three rules that came from mistakes

No function, package or file names in the spec. The early drafts described behaviour through the code, along the lines of "`ListDelegations` returns...". A spec written like that breaks with any refactor even when the behaviour stays the same, and someone from QA can't review it. Now the spec talks about the observable contract (proto fields, flags, domain states), and the names go in `design.md`, which is allowed to change with the code.

Don't document defective behaviour we keep on purpose. Sometimes a change leaves an odd behaviour in place because fixing it belongs in another change. When Claude described the current behaviour, it wrote those oddities down as requirements with a SHALL. At that point the bug was part of the contract, and whoever fixed it later would be "breaking the spec".

Map every scenario to the test that covers it. Scenarios lived in the specs repo, tests lived in the service, and nobody checked one against the other. Now `design.md` ends with a table, scenario on the left and test on the right. The rows I care about are the ones that say "no dedicated test". In our second change two rows said that, covering six scenarios, and they were written down in the design instead of being found later.

## The rest is convention

The other rules didn't come from a specific mistake. They keep changes looking alike:

- Prose in Spanish, but OpenSpec's headers, SHALL/MUST and GIVEN/WHEN/THEN in English, because OpenSpec parses them.
- Requirements titled `Requirement: REQ-NN <title>`, so a review comment can just say "REQ-02".
- One scenario per observable rule.
- The proposal's Impact names the affected services and says whether a contract changes and who will notice.
- `tasks.md` includes a ticket and a PR per repo, QA in dev, and archiving after the production deploy. That last one gets its own post.

## Impact

- **Changes share a shape.** Two changes is a small sample, but the second one, written with the config, can be reviewed with the same checklist as any change that comes after it.
- **Specs that QA can read.** With no function names, scenarios can be reviewed without opening the code.
- **Test gaps written down before merge.** The scenario to test table in the second change showed six scenarios without a dedicated test, in the design and in plain sight.

## Decisions

- **Rules in `config.yaml` instead of CLAUDE.md.** The config lives in the specs repo and OpenSpec applies each block to the artifact it belongs to. A CLAUDE.md applies to everything Claude does in that folder, and an ad hoc prompt depends on whoever writes it.
- **Start empty and add rules after real problems.** We didn't try to write the full rulebook up front. The three rules above came from drafts that got them wrong.
- **Two languages on purpose.** Spanish for prose because that's how the team writes and reviews. English for the parts the tool parses.

## Limitations

- **The rules guide, they don't enforce.** `openspec validate` checks structure. It won't tell you that a scenario names a function or blesses a bug. A person still checks that in the spec PR.
- **Some rules need knowledge the AI doesn't have.** "Don't document defective behaviour" only works if someone has said which behaviour is defective. That has to come up in explore, or in review.
- **Old changes stay as they were.** The first change, already archived, doesn't follow any of this.

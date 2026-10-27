---
title: "We wrote a spec for code that already worked. Two tests failed"
date: 2026-12-08 00:00:00
lang: en
ref: openspec-refactor-spec
tags: [openspec, spec-driven, refactoring, testing, claude-code]
read_min: 6
excerpt_text: "We wanted to merge two ways of reading the same data in one service without changing what consumers see. Before touching code we wrote an OpenSpec spec of the current behaviour and turned its scenarios into tests against a real database. On develop, two of them failed. Nobody had reported either bug."
---

One of our services had two ways of reading the same tree of data, and they disagreed. We wanted to merge them into one without changing what any consumer receives. The risk in a refactor like that is breaking a behaviour nobody wrote down, so before touching the code we wrote it down. We wrote an OpenSpec spec of how the service behaves today, turned each scenario into an integration test, and ran them against the current code. Two failed. Those were two bugs nobody had reported.

This is the third post on how we use OpenSpec as a team. The [previous one](/openspec-config-rules-for-ai-specs/) covered the rules we give the AI. This one is about using a spec when nothing is supposed to change.

## The code

`accounts` stores delegations: an account can manage other accounts, and those can manage others, so it forms a tree. Several RPCs and events expose that tree. There were two ways of reading it:

- One went through the ORM, read a single level, had optional filters and paginated with a maximum of 100 per page.
- The other was hand-written SQL with fixed filters, did the recursion inside the repository and returned only IDs.

Each caller used one or the other, and neither had tests against a real database. The plan was a single one-level read with combinable filters, with the tree traversal and pagination moved up to the service layer.

## Spec first

We created the change as a new capability, `account-delegations`. The spec describes the tree as the RPCs and events expose it: proto fields, flags, domain states. Following the [config rules](/openspec-config-rules-for-ai-specs/), it has no function names, so it survives the refactor it's there to protect.

One of the requirements, in short:

```markdown
### Requirement: REQ-02 Managed accounts

When an account's delegations are requested, the system MUST return in
`delegations.managed` the accounts it manages directly and, in addition,
the accounts each of those manages. Always two levels.

#### Scenario: Second level of several managed accounts
- **GIVEN** an account X that manages A and B
- **AND** A manages A1
- **AND** B manages B1, all valid
- **WHEN** GetAccount is called for X with `with_delegations` = true
- **THEN** `delegations.managed` **SHALL** contain A, B, A1 and B1
```

The first task in `tasks.md` was to write integration tests from those scenarios, against a real database in a container, and run them on develop before changing anything. The second task was the refactor.

## The two red tests

On develop, every test passed except two.

The scenario above returned A, B and B1. With more than one directly managed account, the second level only kept the accounts of the last one.

The other one was a scenario with 101 managers. The RPC that lists them returned 100. It went through the paginated read, which stops at 100 per page, and nothing asked for the next one.

Nobody had reported either. With small trees and fewer than 100 managers, which is almost every account, both behaved correctly.

## What we did with them

We fixed them in the same change and said so. The proposal lists both under What Changes. Impact names the consumers that will start receiving more IDs (`billing`, `reports` and a cache in `gateway`) and states that the proto doesn't change. The rest of the behaviour is pinned by the spec, and the refactor had to keep it green.

`design.md` maps each scenario to its test. Two of the scenarios have a test of their own because of the bugs: one for the second level across several managed accounts, one for more than a page of managers. The mapping also shows two rows with no dedicated test, and they're written down there.

## Impact

- **Two bugs found before anyone reported them.** Both would have survived the refactor, because the refactor was meant to keep behaviour as it was.
- **The refactor had a safety net.** The new read and the tree code went in with the old behaviour pinned by tests against a real database, which the service didn't have before.
- **Consumers were told in advance.** The behaviour change is in the proposal, with the list of services that receive more IDs, before any code was merged.

## Decisions

- **Spec and tests before the refactor.** A refactor spec describes the current behaviour, so it can be checked against the current code. That check is what found the bugs.
- **A real database instead of mocks.** One of the two reads was hand-written SQL. A mock would have returned whatever we told it to, and both bugs lived in how the queries were combined.
- **Fix the bugs in this change, in the open.** The other option was a spec that described the bugs, tests asserting them, and a second change to flip both. Our rule is not to document defective behaviour, so the fixes went in here, listed in the proposal.

## Limitations

- **The spec covers the read paths only.** Two write operations also walk the tree, and they were left out of this change. Their behaviour is still unwritten.
- **Writing the spec of something that exists takes time.** Each scenario needs someone who knows which behaviours are intended, and that knowledge is usually spread across several people.
- **Not every scenario has a test.** The mapping shows six scenarios without a dedicated one. They're visible, but they aren't covered.
- **More IDs is still a change.** It's the correct behaviour, but a consumer could depend on the old output. We wrote that down as a risk in the design rather than verifying every consumer.

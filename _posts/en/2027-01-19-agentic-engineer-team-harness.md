---
title: "Chasing the AI doesn't scale: an agentic engineer's job is the team's harness"
date: 2027-01-19 00:00:00
lang: en
ref: agentic-engineer-team-harness
tags: [harness, spec-driven, agentic-engineering, workflow]
read_min: 15
excerpt_text: "A lot of teams now generate twice as much code with AI and still review it line by line. The senior ends up stuck reading thousand-line PRs while everyone else waits. What changed is that specs, contracts and tests, the first things we used to cut, are now cheap. Here's how we're building a team harness, and why I think knowing how to design it is the skill that defines an agentic engineer."
---

When I talk to people at other companies about how they use AI to write code, I keep hearing the same story. The team ships far more code than it did a year ago. PRs have grown. And review still works the way it always did: someone senior opens the diff and reads it top to bottom.

That was fine with 200-line PRs. With 2,000-line PRs, several at once and all of them looking reasonable, the senior becomes the bottleneck. Either they spend the day reviewing while the rest of the team waits, or they start skimming and the bugs reach production. I've heard both versions, and in both the conclusion people draw is usually the wrong one: the AI writes too much code, or we need to review more carefully.

I think the problem is somewhere else. Those teams use AI to write faster and left the rest of the system as it was. They're chasing the AI, running after whatever it produces. On my team we've spent the last few months trying the opposite: using AI to build the system the whole team works in, so that reading thousands of lines stops being the place where we decide whether something is right.

This post is longer than usual because it pulls together pieces I've written about separately. It's also a pilot, and I'll say that up front: we're trying it out and changing it as we go.

## The bottleneck moved

For years the cost of a change was in writing it. Thinking through the design, writing the code, fighting with the tests. Review was a small slice of the total.

Writing the code is cheap now. An agent gives you a new handler with tests in a few minutes. The expensive part is knowing whether what came out is correct, and we're still trying to answer that with the same tool as always: a person reading the diff.

I think that's why so many companies aren't seeing the productivity gains they expected. Development really is faster. But if the way the team works stays the same, the time saved on writing code doesn't disappear. It moves to other parts of the process, mostly the review queue and chasing the bugs that slip into production. Measure it from ticket to production and the improvement is a lot smaller than the demo promised. Without a harness strategy for the whole team, AI speeds up the part where code gets written and the rest of the path stays just as slow.

Reading an AI-generated diff is also harder than reading a teammate's. I covered this in [the post on reviewing AI code](/reviewing-ai-generated-code-before-you-trust-it/). A junior's code tends to be obviously wrong, while the AI's is plausibly wrong. It reads clean, the tests pass, and the bug sits in a handoff between services the model couldn't see. Catching that takes close attention, and nobody holds that attention across 2,000 lines.

So the senior who reviews everything doesn't scale. Asking every developer to review harder doesn't scale either. What can scale is moving verification to places where it's cheaper and doesn't depend on someone reading every line.

## What we couldn't afford before

Every team I've worked on had things we knew we should do and never quite did. Write the spec before the code. Pin down the contract between services before the consumer started depending on it. Have a test for every behavior that matters, and not only for the functions that were easy to test.

We skipped them because they took time, and the time went into writing code. When a deadline got close, they were the first thing to go. The spec stayed in the ticket description, the contract got adjusted one incident at a time, and the tests covered whatever there was time for.

That's exactly what changed. A first draft of a spec with its scenarios now takes minutes. Same for a contract change in a proto, or a batch of tests from a list of behaviors. The draft still needs reviewing, of course. But reviewing a 60-line Markdown spec is much cheaper than reviewing 2,000 lines of Go, and it happens earlier, when changing something costs one line of text.

Here's how I see it now. I don't want to spend the time AI saves me on writing code by reading more code. I want to put it into everything we couldn't afford before, because that's what later lets you verify the code without reading all of it.

The [Agentic Engineer Starter course by LIDR](https://lidr.notion.site/curso-agentic-engineer-starter) helped me put a name to this. The rest of the post is how we're applying it on my team.

## The team harness

In [the harness post](/fix-the-harness-not-the-prompt/) I defined it as everything around the model: the instructions it loads, the tools, the permissions, the hooks, and what it sees when something fails. That post was about my harness, the one in my own sessions. This one is about the team's, which is what actually changes review.

These are the pieces we have today, in the order they show up during a change. Service names are made up.

### The spec, reviewed before the code

We use OpenSpec with [a central repo just for specs](/openspec-one-repo-for-all-specs/). Every change starts with a proposal and scenarios, generated with Claude from the ticket and then reviewed in a PR. The developers of the affected services open that PR, and so does product. Product doesn't read Go, but they do read scenarios written as observable behavior.

The spec PR is a gate. Nobody writes code until it's approved. It's the cheapest place to find out that someone understood something different, or that a case is missing, because fixing it there means editing a paragraph.

To keep AI-generated specs consistent, the whole team uses the same `config.yaml` with rules per artifact type. I went through it in [the config post](/openspec-config-rules-for-ai-specs/). Several of those rules came from real failures, and I'll come back to them below.

### One scenario, one test

An approved spec isn't worth much if nobody checks that the code meets it. One of the config rules says `design.md` has to end with a table mapping each scenario to the test that covers it.

Here's a made-up example in the format we use. The scenario:

```markdown
### Requirement: REQ-03 Idempotent refund

#### Scenario: Retrying the same refund
- **GIVEN** an order in `orders` charged for €50
- **AND** a refund already accepted with `idempotency_key` = "r-1"
- **WHEN** another RefundOrder arrives with the same `idempotency_key`
- **THEN** the response **SHALL** return the original refund
- **AND** `billing` **SHALL** hold a single refund for that order
```

The test that comes out of it, an integration test against a real database in a container:

```go
func TestRefundOrder_RetryWithSameKey(t *testing.T) {
	ctx := context.Background()
	env := newTestEnv(t)
	order := env.chargedOrder(t, 5000)

	req := &ordersv1.RefundOrderRequest{
		OrderId:        order.Id,
		AmountCents:    5000,
		IdempotencyKey: "r-1",
	}

	first, err := env.orders.RefundOrder(ctx, req)
	require.NoError(t, err)

	second, err := env.orders.RefundOrder(ctx, req)
	require.NoError(t, err)

	assert.Equal(t, first.RefundId, second.RefundId)
	assert.Len(t, env.billing.Refunds(t, order.Id), 1)
}
```

And the table in `design.md`:

| Scenario | Test |
|---|---|
| REQ-03 Retrying the same refund | `TestRefundOrder_RetryWithSameKey` (integration, `orders`) |
| REQ-03 Same key, different amount | no dedicated test |

The rows I care about most are the ones that say "no dedicated test". Those gaps used to turn up in production, or never. Now they're written down in the design, and the reviewer can decide whether to accept them or cover them.

This changes what I do when I review the code PR. I no longer read 2,000 lines asking myself whether they do what they should. I check that every scenario has its test, that the tests really exercise the scenario (as I wrote in [the post on green AI tests](/testing-with-ai-green-doesnt-mean-tested/), a generated test can pass without proving anything), and that whatever is left out is written down. Then I read carefully the parts where the risk is in the design. There usually aren't many.

### Coverage from the ticket (pilot)

This is the newest piece and the least battle-tested. It's a command you run on a ticket once development is done. It reads the ticket and the branch in every affected repo, follows the calls from the changed code beyond the diff, and classifies each behavior as new or as a possible regression.

From that it produces three things and posts them as comments on the ticket:

1. A coverage matrix: what needs manual testing and what doesn't, with the reason. New functionality always goes on the test list. Regressions already covered by passing tests go on the other list, naming the tests that cover them.
2. Test suggestions. For each gap, the smallest test that would cover it and what kind it would be (unit, integration or end-to-end).
3. A test plan with concrete cases, in a spreadsheet linked from the ticket.

Everything runs on read-only worktrees of the branch, so it never touches the repo someone is working in.

What I like about this piece is that it takes the harness beyond the code. Whoever validates the ticket doesn't have to rebuild what changed by reading the diff. They get a list of what to look at, and the part already covered by tests comes with a justification.

What I don't know yet is how far we can trust the matrix. The risk that worries me most is that it marks a regression as covered because some passing test goes through that code, even though it doesn't check the specific case. So for now we treat it as a draft that gets reviewed, and a person looks at the test plan before it's used.

### Reviewers before people

Before asking a teammate for review, the branch goes through automated reviewers. They're read-only agents, each with its own focus:

- One reviews the branch diff against main and returns findings with severity and `file:line`, or an explicit approval if it finds nothing.
- Another runs a security pass on the diff.
- Another compares the code with the change's design doc and flags where it drifts from what was agreed.

None of them replaces human review, but they take work off it. Mechanical bugs, like an ignored error or a new field that never gets propagated, usually arrive already flagged, and the reviewer can go straight to what only a person can judge. It's the same idea as the [writer/reviewer pattern](/claude-code-writer-reviewer/), applied as a fixed step in the flow instead of something each person decides whether to do.

### Rules that don't depend on someone remembering

The last layer is the rules that must always hold. The example from the harness post is `vendor/`: the agent sometimes hand-patched a vendored dependency, the patch passed the tests and then vanished on the next `go mod vendor`. The rule went from CLAUDE.md to a skill, and from the skill to a hook that blocks the write.

That hook still only lives on my machine, and that's exactly the step it's missing. A harness rule only I have protects me, but the next teammate who runs into the same problem will repeat it.

## Every failure goes into the harness

This is the part that has changed how I work the most. When something goes wrong, fixing the bug no longer feels like enough. If I only fix it, the next developer, or the next agent, can make the same mistake a month from now. So besides the fix I ask which part of the harness let it through, and I take the failure there.

Over time I've noticed each kind of failure has a natural home:

| Kind of failure | Where it goes | Real example |
|---|---|---|
| Someone misunderstood what the system should do | A scenario in the spec | In [the `accounts` refactor](/openspec-spec-before-refactor/), writing the scenarios before touching code surfaced two bugs that had been in production for a while |
| The AI keeps getting a type of artifact wrong | A rule in the spec config | The first specs named functions and packages. Now a rule forbids it and the names go to `design.md` |
| A rule that breaks once in a while and costs a lot when it does | A hook | Hand edits under `vendor/` |
| The agent is missing context or a procedure | A skill or docs | How to change an internal library: sibling repo, `replace`, `go mod vendor` |
| Behavior that broke without anyone noticing | A test, plus its row in the scenario table | "No dedicated test" scenarios that get covered in the next change |

The table isn't a fixed rule. Some failures go to two places at once, like `vendor/`, which has a skill to explain the right path and a hook to block the wrong one. What matters to me is that the question always gets asked, and that the answer ends up in something versioned that reaches the whole team.

It also changes how I see reviews. If I find the same kind of problem in a review for the second time, what I'm looking at is a gap in the harness. A comment on the PR fixes it once. Turning it into a rule, a test or a hook fixes it for everyone after.

## How to tell whether the harness works

All of this can sound like opinion, so the most honest test I can think of is simple to set up: give the AI the same task with the harness and without it, several times, and compare.

The task has to be realistic and identical in both cases, for example the idempotent refund above. Without the harness, the agent gets the code and the ticket text. With it, the agent also gets the approved spec, the CLAUDE.md with the commands that define "done", the scenario-to-test rule and the hooks. Each variant runs N times in non-interactive mode on the same model. Then I look at three things.

### Output quality

So it doesn't come down to my impression, before running anything I write an acceptance test suite from the scenarios, one the agent never sees. Each run's score is how many of those scenarios it passes, whether lint comes out clean, and how many findings an independent reviewer raises.

### Token spend

Every headless run reports input and output tokens, cost and number of turns. I expect the harness to spend more per run, since it loads the spec and runs the tests. The question I care about is what it costs to get to a good result, counting the runs you have to repeat or fix by hand.

### Predictability

This is the one that matters most for a team. If I run the same task five times, I want five results that look alike in quality and in the design they pick. An agent that sometimes nails it and sometimes doesn't forces you to review everything closely, and we're back to a senior reading 2,000 lines. A good harness should narrow that spread, because it takes away decisions the model had no business making.

I don't have the numbers for this comparison yet. When I do I'll write them up, whatever they say. In the meantime it works as a yardstick: every piece we add to the harness should improve at least one of these three things without making the others much worse.

## Agentic engineering as a skill

I've been thinking for a while about what separates someone who uses AI well from someone who can get a whole team to use it well. I think it comes down to the level of abstraction they work at.

AI is a tool, like the IDE or CI were. A developer who uses it well writes faster. That's worth something, but the gain stays with one person. And if the rest of the system stays the same, that extra code ends up in someone else's review queue.

When I say agentic engineer I mean a skill. I don't see it as a new box on the org chart, or as one person who does only this. It's the ability to think about the system the team works in, on top of the code you're writing today. Whoever has it designs the harness: where the spec lives and who reviews it, what rules the AI follows when generating it, how tests are derived, which reviewers run before a person does, what a hook blocks. And they design it so it works the same for someone who joined yesterday as for someone who's been on the team for years.

Because it's a skill, any developer can build it, and I'd rather have several people on the team who have it. If only one person knows how to improve the harness, we're back to the same bottleneck as before, just somewhere else. Every time someone takes a failure into the harness instead of just fixing it, they're using that skill.

That's why I care so much about distribution. Specs and their config live in a shared repo. Skills, reviewer agents and commands go into [the team's plugin marketplace](/claude-code-team-marketplace/), so a new joiner gets them with one install. A harness improvement that stays in my `~/.claude` helps me. One that lands in the team plugin helps everyone from the next day on.

My estimate, and it's an estimate because we haven't measured it yet, is that a harness like this can make each person on the team 10 to 30% more productive. That number interests me far more than my own. Me going 30% faster is nice. Everyone on the team doing it, with nobody waiting for a senior to read 2,000 lines, changes what the team can get done.

## Impact

- **Review moved earlier and got smaller.** Misunderstandings show up in the spec PR, where product comments too, and the code PR gets checked against scenarios and tests instead of being read end to end.
- **Coverage gaps are written down.** The scenario-to-test table in `design.md` makes untested cases visible before merge.
- **Three automated reviewers run before a person does**, so mechanical bugs reach human review already flagged.
- **Several harness rules came from specific failures**: three rules in the spec config and the `vendor/` hook. Each one now covers everyone who goes through that layer, not just whoever hit the failure.
- **10 to 30% more per person**, by my estimate. It isn't measured and I don't want to present it as data.

## Decisions

- **Invest in specs and tests before reviewing harder.** Reviewing a huge diff better depends on one person's attention. A reviewed spec and one test per scenario don't.
- **Central spec repo with a shared config.** Changes cross services, and the spec has to be reviewable by people from several repos and from product. A config per repo meant every change came out shaped differently.
- **Read-only automated reviewers as a fixed step.** If each person decides whether to run them, they get run when there's time, which is exactly when they matter least.
- **Each failure goes to the harness layer that matches how costly it is to repeat.** A config rule is cheap. A hook is code to maintain, so I keep those for things that break and hurt.
- **Compare with and without the harness on quality, tokens and predictability.** Looking only at the quality of one run hides how much it cost to get there and how little it may resemble the next one.
- **The coverage flow is a reviewed draft.** Until we know how accurate it is, a person checks the matrix and the plan before they're used.

## Limitations

- **It's a pilot.** We're trying it on the team and changing it as we go. Some pieces, like the coverage flow, haven't been in use for long.
- **We have no metrics.** We haven't measured review time or lead time before and after. The 10 to 30% is my estimate, and the with/without harness comparison is still to be done.
- **Maintaining the harness costs time.** Rules, skills, reviewers and hooks are code and text that need looking after, and someone has to set time aside for it explicitly.
- **Writing a spec for something that already exists is expensive**, because it needs someone who knows which behaviors are intentional.
- **Part of the harness is still personal.** The `vendor/` hook only exists on my machine until we move it into the team plugin.
- **The harness doesn't replace judgment.** Design decisions, service boundaries and trade-offs still get a careful human review. What the harness does is free up time for that.

---
title: "Multi-agent orchestration: when the fan-out pays for itself"
date: 2026-09-15 00:00:00
lang: en
ref: multi-agent-when-worth-it
tags: [multi-agent, agents, cost, workflow]
read_min: 5
excerpt_text: "Discover subagents and the instinct is to fan everything out — ten agents, one per concern, all parallel, fast and thorough. The bill comes back 8x a single pass and half of them re-found the same issues. Fan-out isn't free parallelism; it's N context windows that can't see each other."
---

The first thing you do after discovering you can spawn agents is spawn too many. I had a review task and fanned it out: ten agents, one per concern, all running in parallel — it *felt* fast and thorough. The bill came back roughly eight times a single pass, and half the agents had independently re-found the same three issues. Parallel, yes. Thorough, not really. Mostly expensive.

Fan-out looks like free parallelism. It isn't. It's N independent context windows that don't know the others exist, and that property is the whole story — it's what makes fan-out powerful on the right task and wasteful on the wrong one.

## What fan-out actually costs

Three costs are easy to skip when the agents are running in parallel and it all feels efficient:

- **Tokens don't divide — they multiply.** Each agent pays full freight for its own context. Ten agents is roughly ten times the token bill, not a tenth of the latency for free. Parallel wall-clock, multiplied cost.
- **Coordination is real work.** Someone has to split the task and *merge the results* — and the merge is the hard part: dedup, resolve conflicts, synthesize. That step doesn't disappear because you parallelized the easy part.
- **Each agent is blind to the others.** They overlap (the re-found issues), they miss the seam *between* their slices that no single agent owned, and they can't build on each other's findings. Independence is the feature and the limitation at once.

## When it pays for itself

Fan-out wins when the work genuinely decomposes into slices that are **independent** (no agent needs to see another's work), **substantial** (each slice dwarfs the coordination overhead), and better served by **breadth than depth** (coverage or diverse perspectives, not one deep coherent thread). Reviewing twenty files that don't reference each other; searching a large space several different ways at once; generating N independent attempts so you can judge between them. The honest test: *would a single agent's context get polluted or blow past its budget doing this serially?* If yes — if the work is too wide or too noisy for one window — fan out, and have each agent return a [lean summary instead of a transcript](/claude-code-subagents/). If no, you're paying the multiplier for nothing.

## When one agent wins

Sequential, dependent work — where each step needs the result of the last — is not a fan-out, and forcing it into parallel agents just manufactures a merge problem you didn't have. A chain of dependent edits, a task small enough that splitting it costs more than doing it, anything that needs a single coherent line of reasoning held in one context: one agent. The default should be one agent; fan-out is what you reach for when one agent demonstrably can't hold the work, not the other way around.

## The shape that actually works

When fan-out *is* right, the shape that pays off isn't "ten agents, go." It's a small pipeline:

- **Decompose by genuine independence**, not by however many concerns you can name — slices that actually don't need each other.
- **Tier the work** — a cheap model does the wide pass, the expensive tier is reserved for synthesis and judgment, the same [match-the-tier-to-the-task](/picking-the-right-model-for-the-task/) logic applied across agents.
- **Budget for the merge.** Make dedup and synthesis an explicit step with an owner (often one final agent), because the coordination cost is real whether or not you planned for it.
- **Verify, don't trust the loudest.** Have findings checked rather than believing whichever agent stated its conclusion most confidently.

## Impact

- **The bill matches the work.** Fan-out is spent only where slices are genuinely independent and wide; dependent or small work stays on one agent, so you stop paying the N× multiplier for serial tasks.
- **Coverage improves where breadth helps.** Twenty independent files or several search angles get real parallel coverage — the case fan-out was actually built for.
- **The merge stops being an afterthought.** Treating dedup and synthesis as an explicit step is what turns ten overlapping outputs into one coherent result.

## Decisions

- **Default to one agent; fan out on proven need.** The trigger is "one context can't hold this without polluting or overflowing," not "I could parallelize this."
- **Only fan out genuinely independent slices.** If the agents need to see each other's work, they shouldn't be separate agents — you'll pay in overlap and missed seams.
- **Make synthesis a named step.** Budget tokens and an owner for the merge; unplanned, it's where the cost and the inconsistency hide.

## Limitations

- **The overlap doesn't fully disappear.** Even well-split slices re-discover some shared context; you can reduce the waste, not eliminate it.
- **The seam between slices is nobody's job.** A problem that lives *between* two agents' scopes is the classic fan-out miss — the synthesis step has to look for it deliberately.
- **It's easy to fan out to feel productive.** Ten agents running is a satisfying sight that can mask a task one agent would have done cheaper and more coherently — the spawn count is not the win.

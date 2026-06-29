---
title: "Designing a service: use AI as an interlocutor, not an author"
date: 2026-09-08 00:00:00
lang: en
ref: design-with-ai-interlocutor
tags: [architecture, design, distributed-systems, workflow]
read_min: 5
excerpt_text: "Ask AI to design your service and it hands back a confident three-box diagram that quietly settled the boundary, the sync/async call, and the delivery guarantee — decisions you never made. A generated design is the worst place to spend its confidence. Use it to attack your design instead."
---

I was splitting a `reports` capability out of `orders` and asked the model to "design the reports service." Back came a clean three-box diagram: a queue, a worker, a cache. Confident, plausible, the kind of thing you'd put on a slide. Then I looked at what it had decided *for* me — where the service boundary fell, that the path was async, that delivery was at-least-once — three of the hardest decisions in the whole design, settled silently and presented as done. The design wasn't wrong, exactly. It was unexamined, and now it had the gloss of a finished artifact.

The hard part of a new service was never the code. It's the boundary, the sync-vs-async call, what happens when a dependency is down for thirty seconds. A generated design is the worst possible place to spend the model's confidence — because confidence is exactly what those decisions don't deserve until you've stress-tested them.

## A generated design hides the decisions

The value of design work lives in the decisions and the *why* behind them. A generated architecture hands you the artifact while skipping the reasoning, so you inherit choices you never weighed — and you find out which ones in production, when the assumption the diagram quietly made turns out to be false. Worse, the model pattern-matches to a generic shape (queue + worker + cache fixes everything) rather than to *your* load, *your* consistency needs, *your* failure tolerance. It looks like your design. It's the average of a thousand other services' designs.

## Use it to pressure-test, not to produce

So flip the role. Bring *your* sketch — your boundary, your call pattern — and point the model at it as an adversary, the same no-stake skeptic stance as the [writer/reviewer pattern](/claude-code-writer-reviewer/):

```text
Here's my design for splitting reports out of orders: [sketch].
Don't propose an alternative. Attack this one. Where does the
boundary leak? What breaks when billing is down for 30s? What
does it assume about ordering that I haven't guaranteed?
```

The model is far better at finding the hole in a design than at choosing the right design. Asking it to attack yours plays to that — and keeps the decisions where they belong. This is the same instinct as [letting it grill you on the plan](/claude-code-grill-me-plan-mode/): the value is in the interrogation, not in a fresh answer.

## Make it argue both sides

When you're genuinely stuck between two approaches — a synchronous call to `billing` versus an event the worker consumes — don't ask "which is better." You'll get a confident pick that's really a coin flip dressed as analysis. Ask it to make the *strongest case for each*, and to name the failure mode each one accepts. Now the model is widening your option space and surfacing the trade-off you'd otherwise have discovered the hard way, instead of hiding the choice inside a recommendation. The decision stays yours; the input got better.

## Keep the pen, write the decision down

You own the boundary and the trade-off. The model is a rubber duck that talks back — fast, well-read, and genuinely useful for finding what you missed, but not the one accountable for the system at 3 a.m. So keep the pen, and when the decision lands, write it down *with* the alternative you discarded and why. That record — the same shape as the **Decisions** block at the bottom of every post here — is what lets the next person, including future-you, see the reasoning and not just the boxes.

## Impact

- **Decisions get made on purpose.** The boundary, the call pattern, and the failure behavior are chosen and recorded, not inherited from a diagram that assumed them.
- **Failure modes surface at design time.** "What breaks when `billing` is down for 30s" gets asked over a whiteboard, where it's cheap, instead of in an incident, where it isn't.
- **The option space widens.** Made to argue both sides, the model surfaces the approach — and the trade-off — you wouldn't have weighed alone.

## Decisions

- **Bring a design to attack, don't ask for one.** The model finds holes better than it picks architectures; use it for the thing it's good at.
- **Forbid the recommendation when you're choosing.** "Make the case for each, name what each one sacrifices" beats "which is better" — the latter hides a coin flip as a verdict.
- **Record the decision and the discarded alternative.** A design you can't explain later is a design you'll relitigate; the *why* is the durable part.

## Limitations

- **It doesn't know your real constraints.** Load, latency budgets, team shape, the legacy quirk in `orders` — the model only knows what you tell it, and a pressure-test against missing context misses the real risks.
- **A good interrogation can still rubber-stamp a bad premise.** If your sketch's core assumption is wrong, the model may attack the details and leave the flawed foundation standing — it critiques what you show it.
- **Confidence is not authority.** The model will argue any side persuasively; eloquence about a trade-off is not evidence the trade-off resolves your way. You still have to decide.

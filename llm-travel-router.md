# Case Study: LLM-Based Travel Routing Engine

> **Status:** Private · under NDA · in active development as a product.
> Partner identity, proprietary data, and commercial figures are withheld. This write-up describes the engineering and the design decisions only.

## The problem

Travel planning sits on top of a combinatorial routing problem (origins, destinations, dates, connections, constraints) wrapped in a *natural-language* problem (what the traveler actually wants is fuzzy, contextual, and rarely stated as clean parameters). Classic routing systems are good at the first half and bad at the second; pure LLM systems are the reverse, fluent, but unreliable on hard constraints and prone to inventing options that don't exist.

The goal was a system that takes an open-ended traveler request and returns routing recommendations that are **both relevant and real**, grounded in an actual network of routes, never hallucinated.

## The approach

A hybrid architecture that keeps the LLM in the role it's good at (understanding intent, ranking, explaining) and keeps a deterministic engine in the role *it's* good at (enumerating valid options against hard constraints):

1. **Intent extraction**: the LLM parses the free-text request into a structured constraint set (origin, flexible vs. fixed dates, budget posture, trip style, hard exclusions).
2. **Candidate generation**: a deterministic routing layer enumerates *only valid* options from a real network graph. The LLM never invents routes; it only ever sees a grounded candidate set.
3. **Scoring & ranking**: candidates are scored against the extracted constraints, with the LLM contributing the soft, preference-driven part of the score (fit to stated style/intent) on top of hard objective signals.
4. **Explanation**: each recommendation comes back with a grounded, human-readable rationale tied to the traveler's actual stated constraints.

The design principle throughout: **the LLM proposes and explains; the engine guarantees validity.** That separation is what makes the output trustworthy enough to put in front of a real user.

## Architecture (described)

```
free-text request
      │
      ▼
[ LLM intent extraction ] ──► structured constraints
      │
      ▼
[ deterministic routing layer ] ──► grounded candidate set (real routes only)
      │
      ▼
[ hybrid scorer: objective signals + LLM preference fit ]
      │
      ▼
[ LLM explanation layer ] ──► ranked, explained recommendations
```

Key engineering choices: a strict grounding boundary (the model cannot emit a route that isn't in the candidate set), structured-output prompting for the extraction step, and a scoring function that keeps hard constraints non-negotiable while letting soft preferences move the ranking.

## Outcome and impact

A working routing engine that turns vague traveler intent into grounded,
explained recommendations, reliable enough to be developed toward a real
product. The interesting part wasn't any single model call; it was the
**architecture that makes an LLM safe to trust** on a problem where being
confidently wrong is worse than being silent.

What it changed, qualitatively (specifics withheld under NDA):

- **Eliminated a class of failure.** By construction the system cannot recommend a route that doesn't exist, removing the hallucinated-itinerary risk that blocks shipping a pure-LLM recommender.
- **Made the output defensible.** Every recommendation carries a rationale tied to the traveler's stated constraints, which is what turns "the AI said so" into something a user and an operator can trust.
- **Moved from prototype toward product.** The grounding-and-scoring separation held up well enough to justify continued investment as a commercial product.

## My role and key decisions

I designed and built it end to end. The decisions I'd defend in an interview:
drawing the strict grounding boundary (LLM proposes and explains, engine
guarantees validity), using structured-output extraction so intent parsing is
machine-checkable, and keeping hard constraints non-negotiable in the scorer so
preferences can never override feasibility.

## Production considerations

What taking this to production turns on: latency budget for the extraction and
scoring steps, a fallback when intent extraction is low-confidence, monitoring
the share of requests that return an empty grounded set (a coverage signal), and
versioning the scoring weights so ranking changes are auditable.

## A clean-room analog you *can* read

Because this code is private, I built a separate, fully open project that exercises the same skill set, grounded candidate generation, constraint scoring, and ranking over a real route network, on public data and with no proprietary content:

➡️ **[travel-route-recommender](https://github.com/marc-albert-global/travel-route-recommender)**

## Why the code stays private

It's under NDA and in development as a commercial product. I'm glad to walk through the architecture, the grounding strategy, the failure modes I designed around, and the trade-offs in a conversation.

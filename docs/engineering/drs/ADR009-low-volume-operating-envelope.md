# ADR009 - Low-Volume Single-Steward Operating Envelope

Formalize as a first-class architectural constraint that Wiki targets a **low-volume, single-steward** adjudication envelope: every inbox item and every merge reconciliation is adjudicated by hand, and the design does not target machine-volume automated production. Producer-agnostic intake welcomes automated producers, but their throughput is bounded by the steward's adjudication capacity, which is the system's backpressure.

## Context

The steward-attention model runs through the whole architecture: C002 - Scope Triage adjudicates each item, ADR004 - Merge Reconciliation Steering asks the steward to choose a reconciliation mode per overlapping merge, and C010 - Scaffold has the steward adjudicate each concept during a fork-and-migrate. ADR004 already leans on a volume assumption without naming it as a constraint — it accepts the two-step merge interaction as "acceptable under the current low-volume assumption" and notes that under high overlap volume the merge choice "would be the steward's main effort."

At the same time the architecture is deliberately producer-agnostic and invites *automated* producers, including OKF's own reference agent, to deposit into the Inbox. Machine-volume automated production combined with one steward adjudicating every item and every merge by hand is a structural mismatch. The assumption is load-bearing, yet it lived only inside one ADR's cost discussion — so "automated producers welcome" and "the steward adjudicates everything" could drift into conflict unnoticed. The architecture must state the envelope it is designed for.

## Options

- **Leave the assumption implicit**: keep relying on it inside ADR004's cost analysis. A load-bearing assumption that is not stated cannot be checked, and silently breaks the first time a producer outruns the steward.
- **Build high-volume affordances now** (batch or auto-triage with sampled review): remove the ceiling by not requiring per-item adjudication. Premature — there is no evidence of the volume, and it adds complexity and an auto-integration path the current envelope explicitly excludes (integration is never automatic).
- **State the low-volume single-steward envelope as a first-class constraint and defer high-volume affordances** *(chosen)*: name the envelope the design is valid within, and treat exceeding it as a future capability decision rather than an emergent surprise.

## Decision

Adopt the **low-volume single-steward operating envelope** as a stated architecture constraint. Per-item triage (C002), per-merge reconciliation (ADR004), and per-concept migration (C010) are all valid within this envelope; the steward's adjudication capacity is the system's backpressure, so a producer cannot force material into the corpus faster than the steward can judge it. If a producer's volume ever exceeds that capacity, it is out of the current envelope and calls for a future capability — for example batch triage with sampled review — which is flagged here, not built now. ADR004's "low-volume assumption" is anchored to this record.

## Consequences

- Enabling: the assumption behind ADR004's cost analysis becomes explicit and checkable, and the steward-attention model stays honest about what it depends on.
- Enabling: gives a clear, stated boundary for *when* new capability is warranted, rather than discovering the ceiling by hitting it.
- Cost: high-volume automated producers are not supported; the single steward is the throughput ceiling and a single point of stall — the same expertise-bottleneck exposure O007-RSK002 names, now acknowledged at the architecture level.
- Cost: raising the ceiling later (batch or sampled-review triage) is a deliberate future decision that must preserve the "integration is never automatic" constraint.

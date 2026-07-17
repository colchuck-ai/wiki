# Wiki — Engineering architecture

This is the engineering root for **Wiki**, the curation tool specified in
[`docs/product/README.md`](../product/README.md). The product tree is the
authority on *what* Wiki must do and *why*; this tree covers *how* — the system's
shape, its components, and the decisions that were hard enough to record.

Read the product tree first. This document does not restate the outcomes; it
derives a structure that serves them and points each requirement at the component
that owns it (see the [Requirement → Component map](REQUIREMENT-MAP.md)).

## The shape

Wiki is a **write-side pipeline over one plain-text, version-controlled corpus**,
surrounded by **standing services** the pipeline calls and **read-side capabilities**
consumers use, all governed by a **policy layer** that makes autonomy safe.

```
                         ┌─────────── policy layer ───────────┐
                         │  C001 Charter    C002 Governance    │
                         │  (scope referent)(envelope+escalate)│
                         └───────▲───────────────▲─────────────┘
                                 │ read          │ evaluate/escalate
 producer ─▶ C006 Intake ─▶ C007 Triage ─▶ C008 Integration ─▶ [ concept corpus ]
             & Held-Aside      (decide)     (SOLE WRITER)          plain-text · git
                 ▲                 │            │  ▲  ▲  ▲              │
                 │ hold            │ route      │  │  │  │              │ read
   C012 Coverage │            drive by intent   │  │  │  │        ┌────┴─────────┐
   & Scope ──────┘   C011 Lifecycle ────────────┘  │  │  │        │ C009 Discovery│
   (surveys)         & Retirement                   │  │  │        │  & Index      │
                                                    │  │  │        │ C010 Question │
        standing services the pipeline calls: ──────┘  │  │        │  Retrieval    │
        C003 OKF Conformance   C004 Source Retention ──┘  │        └───────────────┘
        C005 Currency & Provenance ─────────────────────────┘  (everyone reports here)
```

Three ideas carry the whole architecture:

1. **One writer for the corpus.** [C008 Integration](components/C008-integration.md)
   is the *only* component that mutates concept files. Every other component that
   wants the corpus changed — Triage resolving a merge, Lifecycle deprecating a
   concept, Coverage & Scope flagging drift — expresses *intent*; Integration
   executes the edit OKF-conformantly and records what it did. This is the
   engineering realization of *intent-driven authoring / assisted upkeep by intent*
   (O004-R001/R003) and it means **there is no shared mutable state on the corpus**.
   See [ADR001](drs/ADR001-integration-sole-corpus-writer.md).

2. **Autonomy funnels through one governor.** Every autonomous admission or upkeep
   action asks [C002 Governance](components/C002-governance.md) *may I act, or must
   I escalate?* against the delegation envelope, and every escalation lands in one
   steward attention surface. *Manage by exception* (PDR003) lives at this single
   seam rather than being re-implemented in each capability. See
   [ADR002](drs/ADR002-manage-by-exception-seam.md).

3. **Legible + reversible, materialized once each.** Autonomy is safe because it is
   legible and reversible (PDR003). The corpus records **decision-provenance** for
   every action ([C005](components/C005-currency-and-provenance.md), one append-only
   log everyone reports to), a per-claim **grounding signal + review tier** stamped
   at authoring ([C008](components/C008-integration.md)), and nothing is destroyed —
   below-bar material is **held aside** ([C006](components/C006-intake-and-held-aside.md))
   and retired concepts are **marked in place** ([C011](components/C011-lifecycle-and-retirement.md)).
   Where these machine-readable signals physically live is
   [ADR004](drs/ADR004-machine-readable-signal-placement.md).

## The material lifecycle

Every product requirement attaches to a stage of the material's life. No stage is
orphaned (this is the completeness cross-check behind the decomposition):

| Stage | What happens | Components |
|---|---|---|
| **Intake** | Any producer submits raw material; nothing is lost before triage | C006 |
| **Triage** | Scope, duplication, significance → a disposition | C007 (reads C001), routes to C006/C008, escalates via C002 |
| **Author** | Distill into a cited, grounded, cross-linked concept | C008 (calls C003, C004, C005) |
| **Maintain** | Keep it current, on-scope, complete, high-signal | C011, C012, C004 (drift), C005 (currency) — all drive C008 by intent |
| **Discover** | Consumers browse, traverse, and query | C009, C010 |

## Standing services and the policy layer

These are called by the pipeline rather than sitting in it:

- **[C001 Charter](components/C001-charter.md)** — the required, in-corpus scope
  referent. Triage, significance, duplication, and coverage all read it.
- **[C002 Governance](components/C002-governance.md)** — the delegation envelope,
  the `evaluate → act | escalate` decision, and the one steward attention surface.
- **[C003 OKF Conformance](components/C003-okf-conformance.md)** — the *single*
  owner of format knowledge. Wiki **consumes** an external, pinned open format; it
  does not define one. Every writer normalizes/validates through here, so a format
  version bump has a one-component blast radius (O004-R002, O006-R002).
- **[C004 Source Retention](components/C004-source-retention.md)** — the durable
  snapshot **and** the distinct live-origin locator for each citation, plus drift
  computation. One durable-source substrate serves both provenance (O001) and
  fidelity (O009) — they must not fork it (PDR001). See
  [ADR003](drs/ADR003-citation-two-reference-model.md).
- **[C005 Currency & Provenance](components/C005-currency-and-provenance.md)** — the
  append-only provenance log, drift-status, and per-concept currency picture.
  Everyone *reports* dispositions here through a narrow interface; no one writes its
  store directly.

## Read-side discovery

- **[C009 Discovery & Index](components/C009-discovery-and-index.md)** — the
  deterministic navigational structure: the index, the reason-annotated cross-link
  graph, and inbound-link / dangling-link queries (referential integrity).
- **[C010 Question Retrieval](components/C010-question-retrieval.md)** — free-form
  question → relevant concepts, for the consumer who cannot guess the heading
  (O005-R004). Deliberately distinct from the index because it is heuristic and its
  mechanism is open (agent-over-corpus vs a built search index). See
  [ADR005](drs/ADR005-question-retrieval-mechanism.md).

## Engineering invariants

Carried from the product tree and honored by every component:

- **No shared mutable state on the corpus.** One writer (C008); everyone else
  drives it by intent.
- **Machine-readable-first.** Every human-legible signal is a machine-readable field
  first, a rendering second — the consumer is a teammate *or agent*.
- **Deprecation over deletion.** Held-aside (pre-integration) and in-place marking
  (post-integration) are the only removals; genuine purge is a separate explicit act.
- **One durable-source substrate** for provenance and fidelity (C004).
- **Charter is required and load-bearing** (C001); scope always has a referent.
- **Manage by exception** through one governor (C002); autonomy is legible
  (provenance, review tier) and reversible (held-aside, in-place marking).

## Architectural Decision Records

Only decisions that are **hard to reverse, surprising without context, and a real
trade-off** are recorded. The tree is deliberately short.

- [ADR001 — Integration is the sole corpus writer](drs/ADR001-integration-sole-corpus-writer.md)
- [ADR002 — Manage-by-exception as a single governance seam](drs/ADR002-manage-by-exception-seam.md)
- [ADR003 — Citation two-reference model (durable target + live locator)](drs/ADR003-citation-two-reference-model.md)
- [ADR004 — Placement of per-item machine-readable signals](drs/ADR004-machine-readable-signal-placement.md)
- [ADR005 — Question-retrieval mechanism owner](drs/ADR005-question-retrieval-mechanism.md)

## Components

See [`components/`](components/). Each spec covers capabilities, the data it owns,
its frozen interface, its relationships, and success criteria tied to the product
proxy metrics. Inter-component interfaces are frozen in
[INTERFACES.md](INTERFACES.md); the full trace is in
[REQUIREMENT-MAP.md](REQUIREMENT-MAP.md).

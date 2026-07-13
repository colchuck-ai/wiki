# ADR001 - Single OKF Conformance Boundary

Route every corpus read and write through one component — the **C004 - OKF Conformance Adapter** — that owns all knowledge-format concerns, rather than letting each component encode OKF conventions for itself.

## Context

The product defers the on-disk format entirely to the Open Knowledge Format and lists "defining the storage format" as an explicit non-goal: Wiki owns the curation workflow, OKF owns the format. Several requirements are nonetheless format-bound — plain-text corpus (O005-R001), open-format conformance (O005-R002), version-control-native storage (O005-R003), consistent naming and stable identifiers (O003-R003), and applying the format's conventions automatically so no contributor need know them (O007-R002). Meanwhile many components — Integration Authoring, Index & Navigation, Currency Tracking, Lifecycle & Retirement — must read or write concepts to do their jobs. The architecture must decide where format knowledge lives.

## Options

- **Format knowledge in each component**: every component reads and writes files and applies OKF conventions itself. Simplest to start, but OKF's rules are duplicated across the system, an OKF version bump touches every component, and conformance can drift component-by-component — directly threatening O005-R002 and O007-R002.
- **Single conformance boundary** *(chosen)*: one Adapter fronts the corpus; all other components exchange concepts through it and hold no format knowledge. Concentrates format coupling in one testable place at the cost of a mandatory indirection on every corpus access.
- **Shared format library, direct file access**: components link a common OKF library but still read and write files directly. Removes duplication but not direct file access, so conformance and naming enforcement remain per-component and unenforceable at a boundary.

## Decision

Adopt the **single conformance boundary**. The OKF Conformance Adapter is the only component that touches the corpus as files; every other component reasons about concepts and links and goes through the Adapter. This makes "the format is deferred entirely to OKF" a structural fact rather than a convention: conformance, naming, and plain-text/version-control guarantees are enforced in one place, and an OKF version change is absorbed at the Adapter without rippling through the pipeline.

## Consequences

- Enabling: open-format conformance (O005-R002) and convention encapsulation (O007-R002) are enforced at a single chokepoint, so no component or contributor can bypass or drift from OKF.
- Enabling: the format is genuinely swappable at its version boundary — an OKF revision is an Adapter change, keeping Wiki's non-goal of not defining the format intact.
- Cost: every corpus access carries one layer of indirection, and the Adapter is a hot path that must not become a bottleneck.
- Cost: the Adapter concentrates responsibility, so its interface must be expressive enough to serve authoring, indexing, currency, and lifecycle needs without leaking file-level detail back to callers.

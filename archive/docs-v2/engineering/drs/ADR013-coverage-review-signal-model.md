# ADR013 - Coverage review signal model and scope-anchored gap suppression

Fuse three heterogeneous gap signals — dangling links (C005), the declared charter scope (C010), and C009's own self-contained agent scan — into one clustered, multi-signal coverage agenda; degrade gracefully to the dangling-link signal alone when no charter is declared; and suppress deliberate non-gaps through the charter's `# Out of scope` rather than a C009-owned dismissal store.

## Context

O007-R002 - Coverage review requires the product to periodically surface areas within the intended scope that are thin or absent, guarding O007-RSK002 (silent gaps). C009 - Coverage Review owns this. Three facts constrain how:

- The signals available to C009 are heterogeneous. Dangling links (`C005.dangling_links`) are concrete but uninterpreted — C005 reports every unresolved link and explicitly defers the judgment of *why* it dangles to its consumer. The charter (`C010.get_charter`) declares intended territory but is **optional** (O008-R001). An open agent scan can propose gaps neither of the others names, but only relative to some notion of scope.
- The architecture forbids a **second scope authority** — the charter is the single authority both intake triage (C002) and reconciliation (C010) adjudicate against (ADR003) — and forbids **shared mutable state** between components. Every other detect face (C006, C008's `retirement_candidates`, C010's `reconcile`) persists nothing.
- O007-R002 says "thin **or** absent," so coverage cannot be limited to areas where nothing exists.

The open questions were how to combine the three signals, how C009 should behave when no charter is declared, and how a steward stops a deliberately-uncovered area from resurfacing every Survey.

## Options

**Combining the signals:**

- **One clustered, multi-signal finding (chosen)**: gather all three signals, cluster by area, and carry every contributing signal on one finding — the same multi-signal shape as C008's retirement candidate. One item per area, not one per signal. C009's own scan is self-contained (per the [ADR004](ADR004-triage-disposition-model.md) / [ADR005](ADR005-integration-authoring-provenance-and-merge.md) precedent), not a call into C002's overlap machinery.
- **One finding per raw signal**: simpler to produce, but floods the steward with duplicates when a dangling link and an agent proposal name the same area.

**Behavior when no charter is declared:**

- **Degrade to dangling links (chosen)**: keep the `dangling-link` signal (authored intent to a concept is meaningful regardless of whether scope was declared) and suppress the `charter-scope` signal and open agent proposals (no declared scope to anchor "intended" against). C009 runs, narrowed, and never errors.
- **Require a charter**: `coverage_review` returns empty with no charter, mirroring `C010.reconcile`. Rejected — it darkens a whole capability whenever scope is undeclared, discarding the scope-independent dangling-link signal that needs no charter to be meaningful.
- **Infer scope from the corpus**: run open agent proposals against scope inferred from existing content. Rejected — proposals would rest on inferred rather than declared scope, manufacturing confident gaps with no authority behind them.

**Suppressing a deliberate non-gap:**

- **Charter `# Out of scope` (chosen)**: the steward adds the area to the charter (`C010.revise_charter`); C009 reads and respects that boundary and never surfaces a gap within it again. No new state.
- **C009-owned dismissal list**: C009 persists a "won't-cover" record per dismissed gap. Rejected — a second scope store C009 must own and keep coherent with the charter, contradicting the single-scope-authority (ADR003) and no-shared-mutable-state principles.

**Coverage scope:**

- **Thin and absent (chosen)**: surface both areas with no concept (`absent`) and existing concepts that under-cover their area (`thin`), per O007-R002's wording.
- **Absent only**: cheaper, but under-delivers the requirement.

## Decision

`coverage_review` aggregates the three signals into one clustered agenda of coverage findings, each carrying a `kind` (`absent` or `thin`) and the set of signals that surfaced it. C009 interprets dangling links (deciding which represent real in-scope gaps, the judgment C005 defers), and runs its own self-contained agent scan for both absent and thin areas. With no charter declared, C009 keeps only the `dangling-link` signal and suppresses `charter-scope` and open agent proposals, never erroring. A deliberate exclusion is expressed once, in the charter's `# Out of scope`, which C009 honors; C009 keeps no dismissal store and no persisted state of any kind. The charter concept itself is always excluded from gaps.

## Consequences

- Coverage quality tracks charter quality: a sharp charter yields a strong agenda; a vague or absent one narrows C009 toward dangling links alone. Sharpening scope (`C010.revise_charter`) is the steward's lever for improving the agenda — one authority, one place.
- Silencing a false gap requires a charter edit, not a transient per-gap dismissal. This is deliberate and auditable, but it means a steward cannot suppress a single surfaced area without expressing it as a scope boundary. Accepted tradeoff, and the reason no dismissal store exists.
- C009 has no state to migrate, invalidate, or keep coherent with the charter; the agenda is always recomputed from its inputs, so a re-run with unchanged inputs reproduces it.
- C009 and C010 are complementary scope faces against one authority: C010's `reconcile` finds content drifted *outside* scope, C009's `coverage_review` finds scope *not yet reached*. The charter's `# Out of scope` now does double duty — bounding reconciliation and suppressing coverage gaps.
- The charter concept is meta to both faces, consistent with the exclusion ADR003 already requires every consumer to honor.
- C009 encodes no OKF structural literal — its scan reads through C004 — so the one-owner-of-format invariant (ADR001) holds with C009 present.

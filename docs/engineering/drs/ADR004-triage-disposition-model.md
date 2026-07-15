# ADR004 - Triage disposition model

Adjudicate each queued item by **recommend-and-confirm** — C002 - Triage evaluates the admission criteria and recommends a disposition, and the steward confirms or overrides before anything changes — record that disposition and the downstream reconciliation direction by **extending C001's ingestion task record** rather than a separate artifact, and detect overlap with C002's **own exact-hash plus agent semantic scan through C004**, with no dependency on the not-yet-specced C005.

## Context

C002 - Triage is the pipeline's adjudication gate between intake (C001 - Ingestion Queue) and authoring (C003 - Integration Authoring). It fulfills the duplication check (O003-R003), the significance bar (O003-R004), and scope-aware triage (O008-R002). The architecture is firm on posture: **integration is never automatic**, **intent in / work out** (the tool performs the work, the steward supplies judgment), and the corpus is low-volume and single-steward, so per-item hand review is the operating envelope, not a bottleneck to engineer around.

Three questions shaped the component and were not fully settled by the existing specs:

1. Within "admission is never automatic," how much should triage **decide** versus merely **surface**?
2. Where should the disposition — and the reconciliation direction C003 needs — be **recorded**?
3. O003-R003 is about **overlap**, not just byte-identical duplicates; C001's `content-hash` catches only exact matches. How should C002 find semantically-overlapping concepts, and does that couple it to the still-unspecced C005 - Index & Navigation?

## Options

**How triage presents its adjudication:**

- **Recommend + confirm (chosen)**: C002 evaluates each criterion and recommends a disposition with rationale and evidence; the steward confirms or overrides; nothing is admitted without a decision. The tool does the synthesis (serving O004 - Low curation effort) while the steward keeps the gate (honoring integration-never-automatic).
- **Evidence-only**: surface per-criterion findings and make no recommendation; the steward judges every call. Retains maximal judgment but pushes all synthesis onto the steward, working against O004 for no gain in a system where the steward already confirms each item.
- **Recommend + auto-apply clear rejects**: recommend for all, but auto-discard unambiguous fails (e.g. exact duplicates). Rejected — it makes removal automatic and contradicts "surface the overlap for the steward to merge, keep, or discard"; even an exact duplicate is a candidate to steer, not a silent drop.

**Where the disposition and downstream direction live:**

- **Extend C001's task record (chosen)**: C002 writes its findings, disposition, and reconciliation direction into the existing ingestion task, advancing the `status` C001's schema already reserves for triage. One artifact carries an item through intake → triage → authoring, and C001 already names the task file "the intake→triage interface artifact." Cost: C002's output shape lives inside C001's staging format — a small, deliberate coupling C001 anticipated.
- **C002-owned triage record**: a separate artifact linked to the task, leaving C001's task intake-only. Cleaner separation of concerns, but a second staging artifact and a cross-link to keep coherent, for no functional gain in a single-steward pipeline.

**How overlap candidates are found:**

- **C002 self-contained scan (chosen)**: exact matches by C001's `content-hash` plus an agent semantic scan that reads corpus concepts through C004's parse helper. No dependency on the unspecced C005; C002 is buildable immediately after C001, C004, and C010. Cost: it rescans the corpus per triage — acceptable at this scale.
- **Depend on C005 for candidate retrieval**: ask C005 - Index & Navigation to return overlap candidates from its index/graph, reusing navigation and avoiding a fresh scan. Rejected for now — it couples C002 to an unspecced component and pulls C005 onto the critical path ahead of C003.
- **Exact-hash only**: catch only byte-identical duplicates and defer semantic overlap. Rejected — it under-delivers O003-R003, which is about overlap rather than identity.

## Decision

Adopt recommend-and-confirm: C002 evaluates scope (against `C010.get_charter()`), overlap, and significance, and recommends a disposition, which the steward confirms or overrides; only the recorded decision changes a task's `status`, and nothing is admitted, merged, or discarded automatically. Record the disposition and the reconciliation direction (`merge` with a target concept-id, or `keep-new`) by extending C001's ingestion task record and advancing its `status`, so a single artifact carries the item to C003. Detect overlap with C002's own exact-hash comparison plus an agent semantic scan through C004, with no C005 dependency.

## Consequences

- A single artifact carries an item through intake → triage → authoring: C003 reads the admitted task's disposition and `direction`. C001's task schema gains a disposition block — a small coupling it already anticipated by reserving `status` for downstream triage.
- The steward's judgment stays the admission gate while the tool does the evaluation and recommendation, serving O004 without making admission automatic. The recorded `decision`, not the recommendation, is authoritative, so an override is faithfully captured.
- C002 is buildable immediately after C001, C004, and C010, with no C005 dependency. The overlap scan's cost scales with corpus size — acceptable in the low-volume, single-steward envelope, and optimizable later by delegating candidate retrieval to C005 without changing C002's contract.
- Overlap surfaces concrete candidate concept-ids, so a merge is steered against a real target; executing the merge is C003's, keeping the whether/what (C002) separate from the how (C003).
- With no charter, scope drops to `not-evaluated` and triage still recommends on overlap and significance, consistent with ADR003 and C010's optional-charter contract.
- C002 writes only to the staging task, never the corpus, and contains no OKF structural literal (it reads concepts through C004), so ADR001's one-owner invariant holds with C002 present.

## Affected elements

- **C002 - Triage** — the component this decision defines; backlinks here.
- **C001 - Ingestion Queue** — its ingestion task record is extended with the disposition block; backlinks here.
- **C003 - Integration Authoring** — consumes the recorded disposition and reconciliation direction (specced later).
- **C010 - Charter** — read as the single scope authority at intake timing (owned under ADR003).

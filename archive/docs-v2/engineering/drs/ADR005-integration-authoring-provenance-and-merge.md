# ADR005 - Integration authoring, provenance, and merge model

Author each admitted item into the corpus through C004's write face, and settle the three questions C003 - Integration Authoring inherited: land a durable source as a **cited-by-path provenance asset** under `/references/` (not as a concept); satisfy per-claim provenance **concept-level by default, inline per-claim only on multi-source merges**; and execute a merge by **folding into the target concept in place, preserving its id and history**. Discover cross-link candidates with C003's own scan through C004, with no C005 dependency.

> **Superseded in part by [ADR011](ADR011-concept-verb-surface.md):** the `author` entry point with a `keep-new`/`merge` mode is replaced by the `create` and `revise` verbs (and the `merge` composition built from `revise` + `delete`). The provenance (cite-by-path), citation-granularity, and merge-reconciliation decisions recorded below are retained unchanged and are now realized by those verbs.

## Context

C003 - Integration Authoring is the pipeline's write stage between triage (C002 - Triage) and the standing corpus, and the only component that writes concepts. It fulfills cited claims (O001-R001) and intent-driven authoring (O004-R001), and shares reasoned cross-links (O005-R002) with C005 - Index & Navigation. [ADR002](ADR002-intake-staging-and-durable-source.md) explicitly deferred the corpus-side provenance layout "to be settled with C003," and C001's promotion boundary left the physical snapshot move to authoring.

Four questions were open and not settled by the existing specs:

1. Where and how does a promoted durable source land in the corpus, and how is it cited?
2. O001-R001 says the product must give **every claim** a reference, but OKF §8 citations are a document-level numbered list. How literally is per-claim traceability enforced?
3. C002 hands C003 a `merge` direction with a target concept-id. How is a merge executed without breaking referential integrity?
4. Cross-linking needs candidate concepts to link to. Does finding them couple C003 to the still-unspecced C005?

Constraints frame the choices: the corpus stays plain-text and version-control-native; a non-text source can only be a bounded binary asset, never an OKF concept (concepts must be UTF-8 markdown with frontmatter); integration is never automatic; and the system is low-volume and single-steward, so per-item hand review is the operating envelope.

## Options

**Where a promoted source lands and how it is cited:**

- **Cite-by-path provenance asset (chosen)**: promote the snapshot into a `/references/` area as a git-tracked asset (text, or the binary fallback for non-text) and cite it by bundle-relative path; cite an already-version-controlled in-place source at its existing path with no copy. The concept graph stays pure knowledge; the model matches C004's exclusion of declared snapshot assets from concept validation; C006 revalidates the asset by the content-hash baseline C001 captured. No meta-exclusion machinery is needed.
- **Sources as first-class `type: reference` concepts**: wrap each source in an OKF concept under `/references/` and cite that concept. Richer and uniform for graph traversal, but inflates the index, coverage review, and retirement candidacy with non-knowledge — each needing charter-style meta-exclusion in C005/C008/C009 — and cannot cover binary snapshots uniformly (those remain assets). Rejected: cost without proportionate gain at this scale.

**How per-claim provenance is enforced:**

- **Concept-level default, inline per-claim on multi-source (chosen)**: a single-source concept's `# Citations` block covers all its claims (a distilled item traces to its origin); a concept fused from more than one source — the merge case — carries inline numbered markers so a reader can tell which source backs which claim. Satisfies O001-R001 without dense markup where one source backs the whole concept.
- **Always inline per-claim**: the strongest literal reading and most auditable, but heavy authoring and dense prose on every concept, including the common single-source case where it adds nothing. Rejected.
- **Concept-level only**: under-delivers O001-R001 once a concept fuses claims from several sources. Rejected.

**How a merge is executed:**

- **Merge into the target in place, preserve id (chosen)**: fold the new material into the existing target concept-id, union its citations and cross-links, reconcile prose per the steward's note, and keep the target's id and history. No new id is minted, so inbound links stay valid and neither C005 (inbound-link resolution) nor C008 (deprecation) is pulled onto the authoring path. `keep-new` mints a fresh concept instead.
- **New concept + deprecate the originals via C008**: cleaner lineage when a merge is truly a supersession, but it mints a new concept-id whose inbound links must be redirected and pulls C008 onto the path of an ordinary merge. Rejected for ordinary merges — supersession is C008's distinct flow, not every merge.

**How cross-link candidates are found:**

- **C003 self-contained scan through C004 (chosen)**: reuse C002's overlap candidates when present, else run C003's own agent scan reading concepts through C004. No dependency on the unspecced C005, so C003 is buildable immediately after C001, C002, and C004 — consistent with [ADR004](ADR004-triage-disposition-model.md)'s precedent for C002. Optimizable later by delegating candidate retrieval to C005 without changing the contract.
- **Depend on C005 for candidate retrieval**: reuse the index/graph and avoid a fresh scan, but couple C003 to an unspecced component and pull C005 onto the critical path ahead of it. Rejected for now.

## Decision

Adopt cite-by-path provenance: promote each durable source into `/references/` as a git-tracked asset (or cite an in-place source where it already lives) and cite it by bundle-relative path in the concept's `# Citations` block. Enforce provenance concept-level by default, adding inline per-claim markers only when a concept draws on more than one source. Execute a `merge` by folding into the target concept-id in place — unioning citations and cross-links, reconciling prose per the steward's note, preserving id and history — and mint a fresh concept only for `keep-new`. Find cross-link candidates with C003's own scan through C004, reusing C002's overlap candidates, with no C005 dependency. All authoring goes through C004's write face, so no OKF structural literal lives in C003.

## Consequences

- The corpus concept graph stays pure knowledge; provenance lives as cited assets under `/references/`, and C006 revalidates them against the content-hash baseline C001 captured. This settles the promotion boundary ADR002 left to C003: snapshot promotion is a C003 operation reading C001's `durable_ref`, and its path changes once, safely, at authoring.
- Every authored concept traces to its source, and multi-source merges stay legible through inline per-claim markers, delivering O001-R001 without dense markup on single-source concepts.
- Merges preserve referential integrity by construction — the target id is unchanged — so C005's inbound-link resolution is not invoked on an ordinary merge and no link dangles. Supersession remains a separate, deliberate C008 action.
- C003 is buildable immediately after C001, C002, and C004, with no C005 dependency; cross-link discovery rescans the corpus, whose cost scales with corpus size — acceptable in the low-volume, single-steward envelope and optimizable later.
- C003 writes only concepts and their citations and links; the index (C005) and recency and history (C007) are maintained by their owners over that output. Authoring is the first event in a concept's history for C007 to record; whether C003 emits the initial `timestamp` through C004 or leaves the value to C007 is settled when C007 is specced (C004 assigns the `timestamp` value to C007).
- C003 contains no OKF structural literal (all format via C004), so ADR001's one-owner invariant holds with C003 present.

## Affected elements

- **C003 - Integration Authoring** — the component this decision defines; backlinks here.
- **C001 - Ingestion Queue** — its deferred promotion boundary is settled: snapshot promotion into `/references/` is a C003 operation reading `durable_ref`; backlinks here.
- **C002 - Triage** — supplies the admitted disposition and reconciliation `direction` (`merge` target or `keep-new`) that C003 executes.
- **C004 - OKF Conformance** — C003 authors exclusively through its write face (frontmatter, slug/path, citation and link formatting), preserving the one-owner invariant.

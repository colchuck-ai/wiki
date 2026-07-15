# ADR007 - Citation link form for drift revalidation

For a citation to a captured-snapshot source, cite the live `origin` URL as the citation's primary link — not the archived `/references/` path — with the archived asset named as a durable fallback copy in the surrounding prose. C006 - Source Revalidation computes its drift baseline on demand by hashing that archived asset's current bytes; no baseline hash is persisted anywhere.

## Context

C006 - Source Revalidation detects when a cited source has changed since a claim was drawn from it (O001-R003), comparing the live source against the durable reference or captured snapshot. Specifying its interface exposed a gap in the already-committed design: C003 - Integration Authoring's citation (per its own spec) cites the *archived* copy — the promoted `/references/` snapshot path, or the in-place repo path — never the live external locator. C001 - Ingestion Queue's own doc anticipated C006 reading `origin` and `content-hash` from its task record, but nothing links an authored concept back to the C001 task that produced it, and C001's queue was designed as transient staging (discarded on rejection), not a permanent provenance ledger. Resolving "what live thing does this claim need to be checked against" therefore has to happen either through the corpus itself or through a dependency on C001 outliving its stated staging role.

This decision does not need to resolve the analogous baseline question for **in-place** citations (a source already version-controlled inside the corpus's own repo, cited unchanged at its existing path) — git history already gives C006 the content of that path as of any earlier commit with no format change needed, so it is a C006-internal implementation detail, not a cross-component contract change. Only the **captured-snapshot** case (an external origin) needs a decision here, because only that case has no live locator recorded anywhere in the corpus today.

## Options

- **Dual-cite the live origin (chosen)**: for a captured-snapshot source, C003's citation link is the live `origin` URL — an ordinary external citation per OKF §8, which explicitly permits an absolute-URL citation link — with the archived `/references/` path named alongside it in prose as the durable fallback copy. C006 reads the origin straight off the concept via C004's citation-parsing helper; no join back to C001 is needed, and no baseline hash is stored anywhere, since C006 hashes the immutable archived asset on demand each time it revalidates. Cost: a small, scoped update to C003's already-published citation behavior.
- **Provenance sidecar**: C003 writes a small non-concept sidecar file next to each promoted `/references/` asset recording `{origin, captured-at}`. C006 reads the sidecar instead of the citation. Also touches C003, and introduces a second place provenance metadata lives (the sidecar) alongside the citation itself — more moving parts for the same information the citation could carry directly.
- **Reach into C001's queue**: leave C003's citation as path-only; C006 resolves a concept back to its originating C001 task (e.g. by matching the archived asset's content-hash against tasks' recorded `content-hash`) and reads `origin` from there. Requires no change to C001 or C003, but makes C001's queue a de facto permanent ledger C006 depends on indefinitely — in tension with C001's own framing as a staging area outside the corpus whose un-triaged and rejected records are meant to be discarded, and reintroduces exactly the kind of cross-component reach-back C002 and C003 deliberately avoided with C005 (ADR004, ADR005).

## Decision

A captured-snapshot citation cites the live `origin` URL as its primary link; the archived `/references/` asset is named as a durable fallback copy in the surrounding prose, not as the citation link itself. An in-place citation is unchanged — it continues to cite the source's existing repo path. C006 never persists a baseline: for a captured-snapshot citation it hashes the archived asset's current bytes on demand (stable since capture, per C001's immutable-snapshot guarantee); for an in-place citation it reads the cited path's content as of an earlier commit via git history. Both reads happen through the concept and the repository directly — C006 has no runtime dependency on C001 after a concept is authored.

## Consequences

- C003 - Integration Authoring's citation behavior changes for captured-snapshot sources: the citation link is the live `origin` URL, not the archived path. This is recorded on C003's document via [CR001](../../crs/CR001-external-citations-cite-live-origin.md).
- C006 - Source Revalidation becomes fully self-contained, matching the precedent C002 and C003 set (ADR004, ADR005) rather than depending on a materialized structure or another component's transient state: it reads a concept's `# Citations` block through C004 and, for a snapshot citation, fetches the live origin itself.
- No baseline content-hash is ever persisted in the corpus. This follows the on-demand-computation precedent ADR006 set for C005's graph and inbound-link queries: recomputing the archived asset's hash on every call is cheap in the low-volume, single-steward envelope, and avoids a second stored structure that could drift from the corpus it describes.
- A citation now reads slightly differently depending on the source's kind (external URL vs. in-place repo path vs. — after this change — external URL with an archived-copy note). This asymmetry is inherent to the two kinds of durable holding C001 already established (ADR002); it is not a new inconsistency introduced here.
- C008 - Lifecycle & Retirement will consume C006's drift findings as one retirement-candidate signal (O003-R002) when it is specced; this decision fixes the contract (a drift finding keyed by concept and citation, not by C001 task) it will be specced against.

## Affected elements

- **C006 - Source Revalidation** — the component this decision defines; backlinks here.
- **C003 - Integration Authoring** — its citation behavior for captured-snapshot sources changes; see [CR001](../../crs/CR001-external-citations-cite-live-origin.md).
- **C008 - Lifecycle & Retirement** — will consume C006's drift findings as a retirement-candidate signal; to be specced against this contract.

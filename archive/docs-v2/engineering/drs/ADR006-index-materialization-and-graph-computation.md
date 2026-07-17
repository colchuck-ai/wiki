# ADR006 - Index materialization and on-demand graph computation

Materialize `index.md` as a physical, git-tracked, OKF §6 artifact that C005 - Index & Navigation generates and keeps in sync at every directory containing concepts, but compute the cross-link graph and inbound-link lookups **on demand** by scanning the corpus through C004 rather than maintaining a materialized reverse-link index.

> **Superseded in part by [ADR011](ADR011-concept-verb-surface.md):** `reindex` is no longer a terminal call that corpus-writers (C003, C008) must remember to trigger — it becomes an **internal effect** of each write verb, so index drift is eliminated by construction rather than left as a caller obligation to catch via Lint. The storage decision recorded below — materialize `index.md` per directory, compute the cross-link graph and inbound-link lookups on demand — is retained unchanged; only the caller-orchestrated framing of the `reindex` step (in the chosen Index-storage option and the second Consequence) is superseded.

## Context

C005 - Index & Navigation fulfills O005-R001 (navigable index), O005-R003 (referential integrity), and shares O005-R002 (reasoned cross-links) with C003. Two independent storage questions had to be settled before its interfaces could be specced:

1. OKF §6 defines `index.md` as an optional, per-directory listing and explicitly allows either a generated file or an on-the-fly synthesis ("Producers MAY generate `index.md` automatically; consumers MAY synthesize one on the fly when none is present"). Wiki had to pick one for the navigable index (O005-R001).
2. O005-R003 requires surfacing the inbound links to a concept before it is removed or relocated. OKF's cross-link graph is not a first-class stored structure — links are markdown references inside concept bodies (C004 §5) — so resolving "who links to X" requires either a maintained reverse index or a scan.

Both choices set precedent beyond C005: C007 - Currency Tracking will face the analogous materialize-vs-compute question for recency and change history, and the architecture's "no shared mutable state" principle and low-volume, single-steward operating envelope bound both decisions.

## Options

**Index storage:**

- **Materialize `index.md` (chosen)**: C005 generates and keeps a per-directory `index.md` in sync, invoked by writers (C003 after authoring, C008 after deprecation/retirement/relocation) whenever the corpus changes. A consumer without any Wiki tooling — a plain file browser, a teammate cloning the repo — sees navigable structure natively, directly serving O006 portable reuse. Cost: `index.md` becomes a generated artifact with its own write path, and callers that mutate the corpus must remember to trigger `reindex`.
- **Compute on demand only**: never write `index.md`; synthesize the listing at query time, exactly as OKF §6 permits. Simpler — no sync burden, no generated file to keep byte-consistent with the corpus — but a consumer without Wiki tooling sees no navigation aid at all when browsing the raw bundle, which cuts against the portability goal OKF's progressive disclosure exists to serve.

**Graph and inbound-link storage:**

- **On-demand full-corpus scan (chosen)**: `inbound_links` and graph traversal scan concept bodies for links through C004's parse helper fresh on each call. No second structure to keep consistent with the corpus; matches C002's and C003's own established precedent (ADR004, ADR005) of self-contained scans rather than a dependency on a maintained index, and fits the low-volume, single-steward envelope where a full scan is cheap.
- **Materialized reverse-link index**: persist an inbound-link map, updated incrementally as concepts are authored or changed. Faster lookups at scale, but introduces a stateful artifact whose consistency depends on every writer updating it correctly — the kind of shared mutable state the architecture's principles caution against, for a scale where the read cost it would save is not yet a real problem.

## Decision

C005 materializes `index.md` at every directory containing concepts, generated and kept in sync by `reindex`, called by any component that writes, deprecates, retires, or relocates a concept (C003, C008). The cross-link graph and inbound-link lookups (`inbound_links`, `dangling_links`) are computed on demand by scanning the corpus through C004's parse helper on each call; no reverse-link structure is persisted.

## Consequences

- `index.md` is a generated, never-hand-edited artifact; regenerating it with no corpus change must be idempotent and byte-identical, so version-control diffs stay minimal — a concrete success criterion for C005.
- Every writer of a corpus change (C003, C008) takes on a contract obligation to call `reindex` for the paths it touched; C005 does not watch the corpus or poll for changes, consistent with "components read and write concepts directly... they do not share mutable state."
- Inbound-link and dangling-link queries scale with corpus size on every call. Acceptable in the low-volume, single-steward envelope and consistent with C002's and C003's own scan-based precedent; optimizable later (a materialized reverse index) without changing C005's external contract if corpus size ever makes the scan cost material.
- C008 - Lifecycle & Retirement becomes the one component with a real dependency on C005: it must call `inbound_links` before completing a removal or relocation. This is the first genuine (non-optional) inter-component dependency introduced among the undrafted components — C002 and C003 deliberately avoided depending on C005 (ADR004, ADR005), but C008's "never dead-end the graph" obligation cannot be satisfied any other way.
- The materialize-vs-compute split (write the index, but not the graph) sets the pattern C007 - Currency Tracking will be evaluated against when it is specced.

## Affected elements

- **C005 - Index & Navigation** — the component this decision defines; backlinks here.
- **C003 - Integration Authoring** — takes on the obligation to call `reindex` after `author`; backlinks here alongside its own authoring decisions.
- **C008 - Lifecycle & Retirement** — becomes dependent on C005's `inbound_links` before completing a removal or relocation, and on `reindex` after; to be specced against this contract.
- **C009 - Coverage Review** — consumes `dangling_links` as a gap signal; to be specced against this contract.

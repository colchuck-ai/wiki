# ADR006 - In-Corpus Currency

Materialize recency and change history as in-corpus OKF artifacts — a `timestamp` frontmatter field per concept (OKF §4.1) and a `log.md` file per scope (OKF §7) — owned by C007 - Currency Tracking, and never rely on git history as the currency system of record. Git history remains a convenience, not the store.

## Context

O004 (Currency) requires timestamped concepts (O004-R001) and a dated, chronological change history per scope (O004-R002). The inbox storage decision ([ADR003 - Inbox Storage Layout](ADR003-inbox-storage-layout.md)) leaned on version control for this: because nothing in `raw/` ever moves, "each item's git history is its complete lifecycle, reinforcing change-history currency (O004) at no extra cost."

But OKF §3 permits a bundle to be distributed not only as a git repository but as a **tarball or a subdirectory within a larger repository** — both of which strip the bundle's git history. A bundle exercised through the very portability O005 promises would therefore lose its currency record: `timestamp` and change history would be legible in the origin repo and blind everywhere else. Currency must survive O005 portability, so it cannot rest on VCS metadata that portability discards.

## Options

- **Rely on git history**: derive recency and change history from commits. Zero authoring cost, but git history is not part of an OKF bundle when distributed as a tarball or subdirectory, and it is not a plain-text in-corpus artifact — so O004 fails exactly when O005 portability is exercised.
- **Hybrid (git plus partial in-corpus)**: keep some currency in git, some in files. Two sources of truth to reconcile, and it still goes blind on any non-git distribution.
- **Materialize currency fully in-corpus** *(chosen)*: record recency and history as OKF-native artifacts (`timestamp` frontmatter and `log.md`) that travel with the bundle in every distribution mode.

## Decision

Adopt **full in-corpus materialization**, owned by C007 - Currency Tracking. On every authored or updated concept, C003 - Integration Authoring reports the change event; C007 sets or refreshes the concept's `timestamp` (ISO 8601, OKF §4.1) and appends a dated entry to the appropriate scope's `log.md` (OKF §7 form, newest first). Recency and change history are derived from these in-corpus artifacts, which are plain-text, OKF-conformant, and carried in the bundle whether it ships as a git repo, a tarball, or a subdirectory.

This **supersedes ADR003's specific consequence** that git history reinforces O004 currency: the `raw/` inbox's git history remains a convenience for inspecting the inbox lifecycle, but it is not the currency system of record. The inbox storage layout itself (flat directory plus `status` field) is unchanged.

## Consequences

- Enabling: O004 currency survives OKF's tarball and subdirectory distribution modes, so recency and change history stay legible wherever the bundle travels.
- Enabling: reinforces the self-describing-corpus goal (O005-RSK002) — currency needs no external VCS to be understood.
- Cost: C007 must actively author `timestamp` and `log.md` on every change rather than getting history "for free" from commits; the duplication between git history and `log.md` is accepted, with `log.md` as the source of truth and git as convenience.
- Cost: an out-of-band edit that bypasses C003/C007 (hand-editing the corpus) can leave `timestamp`/`log.md` stale; this is out of scope under "intent in, authoring out," where all authoring goes through the tool.

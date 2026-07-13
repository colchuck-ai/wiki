# ADR003 - Inbox Storage Layout

Store C001 - Inbox items as one plain-text file per item under a `raw/` directory, with lifecycle state held in a `status` frontmatter field and nothing ever moved between locations. Keep the component named "Inbox" while naming the on-disk directory `raw/`.

## Context

C001 - Inbox is a Wiki-owned store outside the OKF conformance boundary, so its on-disk layout is Wiki's to define. Two questions needed settling: how item state (`pending`, `integrated`, `discarded`) is represented on disk, and what the store is called. Both are constrained by decisions already made — captured material is retained, never deleted (see [ADR002 - Retain captured source snapshots](ADR002-retain-captured-source-snapshots.md)), and the layout must keep backlog count and age cheap to read (O002-R002) while staying plain-text and version-control-native (O005).

## Options

Layout considered:

- **State-as-subdirectory** (`raw/{pending,integrated,discarded}/`): the folder is the state; a transition is a file move. Trivial `ls`-based backlog, but conflates the transient queue with permanent provenance, and every transition is a git rename.
- **Flat directory + `status` field** *(chosen)*: one file per item; state is a frontmatter field; nothing moves. The integrated file's frozen body doubles as the captured-source snapshot, so a file's git history is its whole lifecycle. Backlog is a filter rather than a bare `ls`.
- **Queue-plus-retention split** (`raw/` queue, `sources/` retained): keeps the queue small but adds a second location and a move per transition.

Naming considered: keep **"Inbox"** vs. rename to **"Raw Data"** vs. **"Data"**.

## Decision

Adopt the **flat directory plus `status` field**. It is the modern email-inbox model — one accumulating store, state as a flag, backlog as the `pending` filter — which matches how the Inbox actually behaves and lets a single file serve as both intake record and durable snapshot, avoiding a second store and any file movement.

Keep the component named **"Inbox"** and name the directory **`raw/`**. A naming panel converged unanimously: components are named for behavior, not the data they hold, and "Inbox" names the capture-and-triage responsibility that "Raw Data" and "Data" do not; "Data" additionally collides with the curated corpus (also data). The directory legitimately holds raw material, so `raw/` is accurate there — behavior at the seam, substance on disk. A component name and its backing directory need not match.

## Consequences

- Enabling: backlog count and age come straight from `pending` items sorted by `captured`, with no separate index to maintain.
- Enabling: nothing moves, so each item's git history is its complete lifecycle, reinforcing change-history currency (O004) at no extra cost.
- Enabling: the raw↔corpus distinction is expressed on disk (`raw/` vs. the corpus) without any change to the product spine's "inbox" vocabulary.
- Cost: the `raw/` directory grows monotonically and backlog reads must filter by `status` rather than trust the directory contents; if volume ever bites, it is a retirement-review concern (O006-R002), not a reason to move files.

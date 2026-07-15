# ADR002 - Intake staging and durable-source capture

Hold un-triaged ingestion tasks in a git-tracked **staging area outside the corpus bundle**, in C001's own non-OKF record format, and durably hold each source **at intake** — referencing it in place when it is already version-controlled alongside the corpus, else capturing an immutable snapshot into staging — then **promote** the snapshot into the corpus provenance area only when the item is authored into a concept.

## Context

C001 - Ingestion Queue is the single producer-agnostic intake point (O007-R001) and owns durable source holding (O001-R002). Two facts shape its storage: un-triaged material must not enter the body of knowledge (integration is never automatic; scope, overlap, and significance are judged later by C002), and a cited source must not be able to rot between intake and the moment it is authored and cited (O001-RSK002). Everything must remain plain-text and version-control-native; a non-text source may be a bounded, version-controlled binary snapshot. The system is low-volume and single-steward, so operational simplicity outweighs throughput concerns.

## Options

**Where the backlog lives:**

- **Staging area, non-OKF (chosen)**: a git-tracked `queue/` sibling to the corpus, each task a plain-text file in C001's own format. Un-triaged material never touches the corpus, index, or graph; C004 governs the corpus only. C001 owns a small task schema.
- **In-corpus OKF tasks**: queue items as OKF concepts in a reserved subtree, reusing C004. Puts un-adjudicated material inside the bundle, risking leakage into the index/graph and blurring "integration is never automatic."
- **Single append-only log**: one backlog file. Simplest, but coarse diffs, awkward per-item snapshot attachment, and messier concurrent edits.

**When/where the durable artifact is captured:**

- **Capture at intake into staging, promote at authoring (chosen)**: snapshot captured next to the task at submit — git-tracked, so durable immediately — then relocated into the corpus provenance area when C003 authors the concept. Keeps un-triaged snapshots out of the bundle; a rejected item's snapshot is discarded with its task, no corpus cleanup.
- **Capture at intake directly into the corpus provenance area**: reference never moves, but seeds the bundle with sources for not-yet-admitted (possibly rejected) items, requiring corpus cleanup on rejection.
- **Defer capture to authoring (C003)**: contradicts C001 owning O001-R002 and reopens the source-rot-before-authoring gap. Rejected.

## Decision

Adopt a non-OKF staging area outside the corpus for the backlog, and capture-at-intake-then-promote for durable sources. C001 records each task and durably holds its source synchronously at `submit` — an in-place path reference when the material is already version-controlled alongside the corpus, otherwise an immutable snapshot (UTF-8 text, or a version-controlled binary asset for non-text) under the task's staging dir, always with a content hash. Durability is thus satisfied the instant the task exists, because the staging tree is git-tracked. On admission, the snapshot is promoted into the corpus provenance area and cited as part of authoring (C003, via C004); on rejection, `discard` removes the staged artifacts and the corpus is untouched.

## Consequences

- Un-triaged material is fully isolated from the body of knowledge; the corpus only ever gains sources for items the steward admits.
- The durable reference is stable within the repo from intake, but its path changes once at promotion. Nothing outside the task points at the staging path before promotion, so the relocation is safe; C003 owns the reference rewrite and citation.
- C006 - Source Revalidation cannot compare a source before authoring — but there is nothing to revalidate until an item is a concept, so this is a non-constraint in practice.
- The content hash captured at intake serves double duty downstream: C006's drift baseline and C002's duplication-check fingerprint. C001 records it without interpreting it.
- C001 owns a small bespoke task format outside OKF; this format is a second (small) place structure is defined, justified by keeping non-corpus staging out of C004's remit. It is deliberately minimal.
- The corpus-side provenance area and its path convention are left to be settled with C003, not fixed here.

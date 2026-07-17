# ADR003 - Charter as an in-corpus concept and single scope authority

Store the charter — the steward's declaration of the corpus's purpose and scope — as a single OKF concept identified by `type: charter`, read and written only through C004's helpers, and make it the single authority both per-item triage (C002) and periodic reconciliation (C010) adjudicate against, at two different timings, under a detect-don't-repair posture.

## Context

O008 - On-scope focus needs a declared anchor for what belongs in the corpus (O008-R001 Charter declaration) that two mechanisms adjudicate against: per-item triage of incoming material at intake (O008-R002, C002 - Triage) and periodic reconciliation of the existing corpus (O008-R003, C010 - Charter). The architecture makes this a principle — "the charter is the single scope authority" — so both mechanisms must consult one artifact, not two drifting copies.

The charter must be **optional** (the corpus works without one), **persistent** for the corpus's life, **revisable in place** (its core purpose can change without becoming a new artifact), **portable** and **version-control-native** (O006), and reachable by C010, C002, and C009. The open questions were where the charter lives, what owns its format, and how out-of-scope content it surfaces is handled.

## Options

**Where and how the charter is stored:**

- **In-corpus OKF concept, `type: charter` (chosen)**: a single conformant concept discovered by its reserved type, authored and parsed through C004 like any other concept. C004 needs no change — it already tolerates unknown `type` values (OKF §4.1) — so C010 adds no format knowledge. The charter travels with the bundle automatically, and revision-in-place is an ordinary edit whose history comes free from git and C007. Cost: it appears in the index and graph as a concept, so consumers must treat it as meta.
- **Wiki-reserved file alongside `index.md` / `log.md`**: naturally excluded from navigation like the other reserved files, but it widens C004 with a third reserved name and charter-structure validation, turning charter shape into a format-owner concern and requiring a C004 change and an amendment to ADR001's boundary.
- **C010-local artifact outside the corpus** (like C001's staging queue): simplest ownership, but the charter would not travel with the bundle — undercutting portability (O006) and the intent that scope declaration lives with the knowledge it governs.

**How out-of-scope content surfaced by reconciliation is handled:**

- **Detect, don't repair (chosen)**: reconciliation surfaces out-of-scope concepts as candidates with a rationale; the steward decides whether to retire (via C008), revise the charter to include them, or keep them. Mirrors C006 and C009.
- **Auto-retire out-of-scope content**: reconciliation deprecates drifted concepts directly. Rejected — it removes the steward's judgment from a scope call and contradicts "integration/removal is never automatic."

## Decision

Store the charter as a single OKF concept identified by `type: charter`, read and written only through C004's parse/serialize helpers. C010 owns the charter's **body semantics** — a required prose `# Purpose` plus optional `# In scope` / `# Out of scope` bullet sections — while C004 continues to own the generic concept format unchanged. The charter is the single scope authority, adjudicated at two timings: per-item at intake by C002, and periodically across the whole corpus by C010's reconciliation. Reconciliation is detect-don't-repair — it surfaces out-of-scope concepts as candidates and never mutates the corpus. The charter is optional; absent it, reconciliation is a no-op and C002 drops only its scope criterion.

## Consequences

- No C004 change and no new format knowledge outside C004: `type: charter` is an unknown-but-tolerated type, so the one-owner-of-format invariant (ADR001) holds. C010 encodes no OKF structural literal; it defines only what the charter's body sections mean.
- The charter travels with the corpus automatically and inherits recency and history from C007 and git, so revision-in-place needs no bespoke versioning.
- Because the charter is optional, the system degrades cleanly: with none declared, C010's `reconcile` returns nothing and C002 still runs its non-scope criteria.
- The charter concept appears in the index and graph. C005, C008, C009, and C010 must treat it as **meta** — never a coverage gap, retirement candidate, or drift candidate. This is a small explicit exclusion every consumer honors.
- Exactly one charter is expected. Multiple `type: charter` concepts are a conflict C010 surfaces for the steward rather than silently choosing one.
- Reconciliation is a whole-corpus agent pass, so its cost scales with corpus size. Acceptable in the low-volume, single-steward envelope; it is surfaced through the product's Survey operation.
- C010 owns the charter and its reconciliation; C002 and C009 are readers, and C008 executes any retirement the steward chooses for a surfaced candidate. Ownership of the scope decision stays with the steward at both timings.

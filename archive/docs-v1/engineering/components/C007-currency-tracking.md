# C007 - Currency Tracking

Records when each concept last meaningfully changed and maintains a dated, chronological history of updates for each scope of the reference, so recency and change history are legible to any consumer — materialized as in-corpus OKF artifacts, never dependent on git.

## Data model

Currency is held in two plain-text, OKF-conformant artifacts that travel with the bundle in every distribution mode (git repo, tarball, or subdirectory — OKF §3):

- **`timestamp` frontmatter** — an ISO 8601 datetime on each concept (OKF §4.1), recording when that concept last meaningfully changed.
- **`log.md` per scope** — a date-grouped, newest-first history of updates for that scope (OKF §7). Each directory level that warrants a history carries its own `log.md`.

Because both are in-corpus artifacts rather than VCS metadata, recency and change history remain legible even when the bundle is distributed with no git history.

## Behavior

Whenever a concept is authored or updated, C003 - Integration Authoring reports the change event. C007 then:

1. Sets or refreshes the concept's `timestamp` to the change time.
2. Appends a dated entry to the `log.md` of the concept's scope, in OKF §7 form (newest first).

Recency and change history are derived from these in-corpus artifacts, not from git history. The corpus is authored following C004 - OKF Conformance conventions rather than through a runtime gateway.

## Edge cases

- **A concept changed several times in one day**: `log.md` groups all entries for that date under a single `YYYY-MM-DD` heading; the concept's `timestamp` reflects the latest change.
- **A scope with no `log.md` yet**: the file is created on the first change recorded in that scope.

## Relationships

- **C003 - Integration Authoring**: reports the change event whenever a concept is authored or updated, including each reconciliation.
- **C004 - OKF Conformance**: the `timestamp` field and `log.md` structure follow OKF conventions (OKF §4.1 and §7).

## Success criteria

- Every concept carries a `timestamp` that reflects its last meaningful change (O004-R001).
- Every scope with a change history carries a dated, newest-first `log.md` (O004-R002).
- Recency and change history remain fully legible when the bundle is distributed as a tarball or a subdirectory with no git history.

## See Also

### Architectural Decision Records

- [ADR006 - In-Corpus Currency](../drs/ADR006-in-corpus-currency.md)

### Change Records

- [CR003 - In-Corpus Currency](../../crs/CR003-in-corpus-currency.md)

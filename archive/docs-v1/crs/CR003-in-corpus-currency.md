# CR003 - In-Corpus Currency

Moved O004 currency off git history and onto in-corpus OKF artifacts, and promoted **C007 - Currency Tracking** from an inline (Tier 0) statement to its own component document. Scoped to the O004 outcome, the C007 - Currency Tracking component, and the ADR003 consequence this corrects.

## Change

- **C007 - Currency Tracking**: promoted from Tier 0 (inline on the architecture document) to Tier 1 (own document). Added a data model — a `timestamp` frontmatter field per concept (OKF §4.1) and a `log.md` file per scope (OKF §7) — behavior (C003 reports each change event; C007 sets the `timestamp` and appends a dated `log.md` entry), edge cases, relationships, and success criteria.
- **O004 (Currency)**: no requirement text changed. What changed is the *realization*: O004-R001 (timestamped concepts) and O004-R002 (change history) are now satisfied by in-corpus artifacts that travel with the bundle, not by git history.
- **ADR003 - Inbox Storage Layout**: its consequence that "each item's git history is its complete lifecycle, reinforcing change-history currency (O004) at no extra cost" is corrected — the `raw/` git history stays a convenience for the inbox lifecycle, but is no longer the currency system of record.

Before, O004 currency partly rested on git history. After, C007 materializes `timestamp` frontmatter and `log.md` in-corpus, so currency survives OKF's tarball and subdirectory distribution modes.

## Rationale

Git history is not part of an OKF bundle when it is distributed as a tarball or a subdirectory (OKF §3), and it is not a plain-text in-corpus artifact. Resting O004 on it meant currency would silently go blind exactly when O005 portability was exercised — the failure mode O004-RSK001 (undetected staleness) and O004-RSK002 (unclear recency) describe. Materializing recency and history as OKF-native `timestamp` and `log.md` keeps them legible wherever the bundle travels.

## Affects

- Product: O004 outcome — realization only; no outcome, risk, or requirement text changes.
- Engineering: C007 - Currency Tracking (new component document and behavior); the architecture Requirement-Component Map and the C007 block and relationships on the architecture document; ADR003 (consequence corrected). Captured by [ADR006 - In-Corpus Currency](../engineering/drs/ADR006-in-corpus-currency.md).

# CR001 - Captured-source retention

Added captured-source retention to the provenance outcome (O001) and specified the corresponding Inbox lifecycle. Introduces requirement O001-R003 and gives the C001 - Inbox component its own document with a defined retention lifecycle. Scoped to the O001 outcome and the C001 - Inbox component.

## Change

- **O001 (Trustworthy provenance)**: added requirement **O001-R003 - Captured-source retention** — the product must retain the captured form of each integrated statement's originating source so the statement stays verifiable even if the external source later changes or disappears. Mapped it as a second mitigation of **O001-RSK002 - Source drift**, alongside the existing O001-R002.
- **C001 - Inbox**: promoted from an inline (Tier 0) statement on the architecture document to its own component document (Tier 1). Added a data model, a three-state processing lifecycle (`pending` → `integrated` snapshot / `discarded` tombstone, with an explicit-purge exception), edge cases, a relationship to C006 - Source Revalidation, and success criteria. Mapped C001 to the new O001-R003 in the architecture's Requirement-Component Map.

Before, the raw inbox item's fate after processing was unspecified, and O001-RSK002 was mitigated only by source revalidation (detection). After, captured material is retained as durable provenance and revalidation has a concrete baseline to diff against.

## Rationale

Source revalidation detects when a source *changes* but does nothing when a source *disappears* — the statement then cites a reference that no longer exists and is unverifiable. Retaining the captured form closes that half of O001-RSK002 and gives revalidation a baseline. Retention also extends the corpus's existing deprecate-over-delete stance (O006) to the point of intake, and rides on O005 portability so it needs no new persistence machinery.

## Affects

- Product: O001 outcome and its Risk-Requirement Map (O001-RSK002 now maps to O001-R002 and O001-R003).
- Engineering: C001 - Inbox (new component document and behavior), the architecture Requirement-Component Map, and the C006 - Source Revalidation relationship. Captured by [ADR002 - Retain captured source snapshots](../engineering/drs/ADR002-retain-captured-source-snapshots.md).

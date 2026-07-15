# C009 - Coverage Review

Supports periodically reviewing which areas of knowledge are represented against the reference's declared scope and other measurable coverage signals, so gaps surface for the steward — who supplies the judgment of what the team relies on (see [PDR003 - Reliance Is Steward-Supplied](../../product/drs/PDR003-reliance-steward-supplied.md)) — instead of persisting unnoticed.

Coverage Review is a standing capability that compares what the reference *has* against what it is *expected* to have, and presents the difference for the steward's judgment. It creates nothing on its own; its output is a **sourcing agenda** — a prioritized list of thin or absent areas the steward acts on by going to *find sources* that then re-enter through Ingest. It runs as part of the **Survey** operation — the scaffold-relative assessment — alongside scope reconciliation (C010 - Scaffold), not as part of the corrective Lint hygiene pass (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Behavior

The hard part is the "expected coverage" input — what the reference *should* cover. Rather than one source, coverage review assembles it from three signals:

1. **Dangling links** — concepts that are linked-to but not yet written. OKF §5/§9 treats a broken cross-link as not-yet-written knowledge, so the corpus itself declares a coverage backlog; these are surfaced via C005 - Index & Navigation.
2. **Scaffold scope** — the scaffold's declared purpose and scope (C010 - Scaffold) frames what the reference is expected to cover.
3. **Agent-proposed gaps** — the Agent Skill can propose areas that appear missing relative to the scope, optionally informed by a web search for what a reference of this kind usually covers.

Coverage review compares the represented-concept directory (C005) against these three signals and presents the gaps for the steward's judgment. It operates within the low-volume, single-steward envelope ([ADR009 - Low-Volume Operating Envelope](../drs/ADR009-low-volume-operating-envelope.md)).

## Edge cases

- **No scaffold declared**: coverage falls back to the dangling-link and agent-proposed signals only, with no declared-scope frame.
- **A dangling link that is deliberately out of scope**: presented to the steward as a candidate rather than auto-created, so an intentional non-topic is not turned into a coverage gap.

## Relationships

- **C005 - Index & Navigation**: reads the represented-concept directory and the dangling-link set that seed two of the coverage signals.
- **C010 - Scaffold**: the declared scope frames what the reference is expected to cover.

## Success criteria

- Coverage review surfaces areas that are absent from the reference relative to its declared scope (O002-R003), drawing on dangling-link, scaffold-scope, and agent-proposed signals, and presents them for the steward's reliance judgment rather than leaving a gap to persist unnoticed.
- The surfaced gaps are expressed as an actionable sourcing agenda — specific enough that a steward can go search for sources to fill each area — rather than an undifferentiated "something is missing" signal.

## See Also

### Architectural Decision Records

- [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)

### Change Records

- [CR008 - Delivery Form and Standing-Capability Specifications](../../crs/CR008-delivery-form-and-standing-capabilities.md)
- [CR010 - Reliance Signal Reframe](../../crs/CR010-reliance-signal-reframe.md)

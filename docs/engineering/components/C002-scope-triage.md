# C002 - Scope Triage

Adjudicates each inbox item against the declared scaffold scope, capturing the steward's keep, merge, deprecate, or discard direction and flagging out-of-scope items for judgment rather than integrating them automatically. When an item overlaps an existing concept, surfaces the overlapping concept(s) so a merge can be steered against a concrete target rather than issued blind.

Scope Triage is the pipeline stage between capture (C001 - Inbox) and authoring (C003 - Integration Authoring). It never writes to the corpus; it produces a steward direction and, for a merge, the set of overlapping concepts the reconciliation must act against. It runs as the triage step of the **Ingest** operation (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Behavior

**Scope adjudication.** Each pending item is evaluated against the scaffold's declared scope (C010 - Scaffold). The verdict — keep, merge, deprecate, or discard — is the steward's; out-of-scope items are flagged for the steward's judgment and never integrated automatically. Adjudication is a scaffold-anchored Agent-Skill judgment, not a fixed algorithm (per ADR010); the scaffold is the single scope authority it reasons against.

**Overlap detection.** Before a merge can be steered, triage must find what the incoming item overlaps. The approach reads the bundle index (C005 - Index & Navigation / OKF §6 `index.md`) to shortlist candidate concepts by title and `description`, then compares the item against the candidates to surface the overlapping concept(s). At moderate scale the index is sufficient and needs no embedding-based retrieval; beyond that, an optional on-device markdown search engine (hybrid BM25/vector with LLM re-ranking) can be shelled out to as a scale affordance rather than a hard dependency.

Triage operates within the low-volume, single-steward envelope ([ADR009 - Low-Volume Operating Envelope](../drs/ADR009-low-volume-operating-envelope.md)): every item is adjudicated by hand.

## Edge cases

- **Missed overlap**: overlap detection surfaces the wrong target or none, and the item is authored as a near-duplicate. This is an O006 retirement-review concern, not a correctness failure — consistent with [ADR004 - Merge Reconciliation Steering](../drs/ADR004-merge-reconciliation-steering.md).
- **Out-of-scope item**: flagged for the steward's judgment rather than integrated; it waits, it does not silently enter the corpus.
- **No scaffold declared**: there is no scope contract to adjudicate against (per C010); triage still captures a keep or discard direction so nothing is auto-integrated blind.

## Relationships

- **C010 - Scaffold**: reads the declared purpose and scope to evaluate each item.
- **C005 - Index & Navigation**: reads the index and cross-link graph to detect which existing concept(s) an item overlaps.
- **C003 - Integration Authoring**: forwards kept or merged items with the steward's intent and, for a merge, the set of overlapping concepts the reconciliation acts against.

## Success criteria

- Every pending item receives a steward direction; out-of-scope items are flagged for judgment rather than integrated automatically (O008-R002).
- When an incoming item overlaps existing concept(s), the target(s) are surfaced so a merge is steered against a concrete target rather than issued blind (supports ADR004).

## See Also

### Architectural Decision Records

- [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)
- [ADR004 - Merge Reconciliation Steering](../drs/ADR004-merge-reconciliation-steering.md)

### Change Records

- [CR008 - Delivery Form and Standing-Capability Specifications](../../crs/CR008-delivery-form-and-standing-capabilities.md)

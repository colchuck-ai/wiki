# C004 - OKF Conformance

The single owner of OKF format knowledge: encapsulates the OKF authoring conventions the curation Agent Skill applies, and provides a conformance validator that checks the bundle against OKF v0.1 §9 and reports violations — so no other component and no contributor needs to know the format's conventions, without every corpus access routing through a runtime gateway.

C004 holds format knowledge; it is not a data path. Components author and read the corpus directly following C004's conventions, and the validator confirms conformance at defined checkpoints rather than gating every access.

## Data model

Reads the pinned specification at [`vendor/okf/SPEC.md`](../../../vendor/okf/SPEC.md) and is the only component that reads it. It exposes two things:

- **Authoring conventions** consumed by the Agent Skill: the frontmatter shape (a parseable YAML block with a non-empty `type`, plus recommended `title`/`description`/`resource`/`tags`/`timestamp`, §4.1); concept identity as the file path minus the `.md` suffix (§2); bundle-relative `/`-rooted cross-links (§5.1); the `index.md` (§6) and `log.md` (§7) forms; and the `# Citations` form (§8).
- **A conformance validator**: given a bundle, it reports the §9 conformance violations — a non-reserved `.md` file lacking a parseable frontmatter block or a non-empty `type`, or a malformed `index.md`/`log.md` when present — and treats OKF's permissive rules as non-violations.

## Interfaces

The validator's contract is OKF §9: it flags only true conformance violations and MUST NOT report OKF's explicitly-permitted conditions (missing optional frontmatter fields, unknown `type` values, unknown additional keys, broken cross-links, or a missing `index.md`) as errors. It is invoked as the conformance check within the `lint` operation and after ingest (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Behavior

Conventions are applied at authoring time by the Agent Skill on every authoring path (C003 - Integration Authoring and any other producer of corpus content); the validator runs at defined checkpoints — after ingest, and as the conformance check inside `lint` — rather than gating each read and write. An OKF version bump (SPEC §11) is absorbed here by updating the conventions and validator in one place; a major bump that changes required fields or reserved filenames is a deliberate change to this component, exactly as the original boundary decision intended, now realized without a gateway.

## Edge cases

- **Broken cross-link**: not a violation (OKF §5/§9 — it may be not-yet-written knowledge). The validator does not flag it; it is surfaced instead as a coverage/navigation signal by C005 - Index & Navigation and C009 - Coverage Review.
- **Unknown `type` value**: tolerated (§4.1) — treated as a generic concept, never rejected.
- **Concept transiently non-conformant between authoring and the next validation run**: caught at the next `lint`/post-ingest checkpoint. This is the accepted cost of enforcing conformance by validation rather than by a mandatory gateway.

## Relationships

- **C005 - Index & Navigation**: the concept and cross-link conventions C004 owns are what index generation and graph traversal rely on.
- **C003 - Integration Authoring**: authors corpus content following C004's conventions directly, rather than writing through a gateway.

## Success criteria

- OKF conventions live in exactly one component, so conformance and naming/identifier rules cannot drift component-by-component (O005-R002, O007-R002).
- The validator flags every §9 conformance violation and none of OKF's permissive non-violations.
- An OKF version change is absorbed by updating C004 alone; plain-text, version-control-native, and naming/identifier guarantees (O003-R003, O005-R001, O005-R003) remain enforced through the conventions and the validator.

## See Also

### Architectural Decision Records

- [ADR007 - Conformance as Conventions and Validation](../drs/ADR007-conformance-conventions-and-validation.md)
- [ADR001 - Single OKF Conformance Boundary](../drs/ADR001-single-okf-conformance-boundary.md) (superseded by ADR007)

### Change Records

- [CR005 - Conformance Boundary Re-scope](../../crs/CR005-conformance-boundary-rescope.md)

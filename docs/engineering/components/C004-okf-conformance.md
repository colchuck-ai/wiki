# C004 - OKF Conformance

The single owner of format knowledge: the OKF authoring conventions the tool applies and a validator that checks the corpus against the spec, so no other component and no contributor needs to know the format.

C004 is a **library and validator**, not a store and not a write proxy. Other components read and write concept files directly on the git-tracked substrate (per the architecture's "no shared mutable state"); they do so *through C004's helpers* so that every OKF structural fact — frontmatter shape, reserved filenames, link syntax, index/log structure, concept-ID derivation, and the Wiki naming convention — lives in exactly one place. C004 binds to a single pinned OKF version ([`vendor/okf/SPEC.md`](../../../vendor/okf/SPEC.md), OKF 0.1) and is the boundary that absorbs OKF version changes, so an OKF bump is a C004 change rather than a system-wide one (see [ADR001](../drs/ADR001-single-okf-conformance-boundary.md)).

## Data model

C004 owns, as data, the format facts every other component would otherwise have to know:

- **Frontmatter schema** (OKF §4.1). Required: `type` (non-empty string). Recommended: `title`, `description`, `resource`, `tags`, `timestamp` (ISO 8601). Producer-defined extension keys are permitted and preserved verbatim. C004 does not register or constrain `type` values.
- **Concept ID rule** (OKF §2). A concept's ID is its bundle-relative file path with the `.md` suffix removed (`tables/users.md` → `tables/users`). C004 derives IDs and resolves them back to paths; other components identify concepts by ID without re-deriving it.
- **Reserved filenames** (OKF §3.1). `index.md` and `log.md` have defined meaning at any level and are never concept documents. Every other `.md` file is a concept.
- **Wiki naming convention.** OKF leaves file naming to producers (§3); Wiki fixes one convention so contributors need not choose. Concept filenames are lowercase kebab-case slugs; C004 derives a slug from a concept's `title` and disambiguates collisions deterministically. Directory topology (how concepts are grouped into subdirectories) stays a producer/steward choice per OKF — C004 constrains slug form and reserved-name avoidance, not the tree shape.
- **Finding model.** A validation result is a list of findings, each `{ severity, rule, concept-id-or-path, location, message }`, where `severity` is `error` (OKF §9 conformance breakage) or `advisory` (Wiki structural profile). Findings describe; they never mutate.
- **Pinned version record.** The single OKF version C004 targets (0.1), sourced from the vendored spec and its pin in [`vendor/okf/PROVENANCE.md`](../../../vendor/okf/PROVENANCE.md).

## Interfaces

C004 exposes two in-process faces. Both are pure of corpus side effects except where a caller uses a returned artifact to write.

**Authoring conventions (the "write" face).** Helpers callers use to produce conformant output while doing the write themselves:

- **Serialize / parse frontmatter** — emit a conformant YAML block from fields (enforcing a non-empty `type` and ISO-8601 `timestamp`, ordering keys deterministically for stable diffs); parse a concept into `{ frontmatter, body }` preserving unknown keys and body bytes.
- **Slug and path** — `slug_for(title)` and `concept_path(dir, slug)`; `concept_id(path)` and its inverse.
- **Reserved-name guard** — reject `index.md` / `log.md` as concept slugs.
- **Index rendering** (OKF §6) — render and parse `index.md` listings. Consumed by C005.
- **Log entry format** (OKF §7) — render date-grouped `log.md` entries. Consumed by C007.
- **Citation block format** (OKF §8) — render/parse a numbered `# Citations` block. Consumed by C003.
- **Link syntax** (OKF §5) — construct bundle-relative (`/…`) cross-links, the recommended stable form. Consumed by C003 and C005.
- **Deprecation marker** — the frontmatter/body convention for marking a concept deprecated or superseded with a replacement pointer. Consumed by C008.

**Validator (the "check" face).** `validate(target) → findings[]`, where `target` is a single concept, a subtree, or the whole bundle. Classifies each violation by severity (below). This is the engine behind the product's **Lint** operation. Read-only.

- Exposes the pinned OKF version so callers and the steward can see what the corpus is checked against.

### Conformance levels

`validate` reports at two severities:

- **`error` — OKF §9 hard conformance.** Frontmatter is unparseable; `type` is missing or empty; a reserved `index.md` / `log.md` is present but malformed against §6 / §7. A non-`.md` or non-UTF-8-text file presented as a concept (the plain-text-corpus floor, O006-R001) — excluding declared source-snapshot assets (see Edge cases).
- **`advisory` — Wiki structural profile.** The *shape* of the fields Wiki's requirements rely on: `timestamp` absent or not ISO-8601; a present `# Citations` block that is malformed; a deprecation marker that is malformed; `okf_version` declared in the root `index.md` that differs from the pinned version. Advisories are structural only — C004 never judges *content* (see Relationships).

## Behavior

- **Permissive parse, tiered validate.** Parsing tolerates everything OKF requires consumers to tolerate (§9): unknown `type` values, unknown frontmatter keys, missing optional fields, broken cross-links. None of these is ever an `error`. Validation is where conformance is judged, split into the two severities above.
- **Round-trip preservation.** `parse` then `serialize` of a conformant concept preserves the body byte-for-byte and every frontmatter key, including unknown producer keys (OKF §4.1), emitting keys in a deterministic order so re-authoring yields minimal version-control diffs.
- **Report, never repair.** The validator only surfaces findings; correcting them is the owning component's or steward's action, mirroring C006's detect-don't-repair posture.
- **Adapter isolation.** Every OKF structural literal resolves through C004. A minor OKF bump adds tolerated fields/headings here; a major bump (renamed required field, changed reserved filename) is absorbed here as a deliberate C004 change, with no edit to any other component (see [ADR001](../drs/ADR001-single-okf-conformance-boundary.md) and `vendor/okf/PROVENANCE.md`).

## Edge cases

- **Captured source snapshot that is non-text**: a version-controlled binary asset held as a provenance fallback (from C001), not a concept — the plain-text-concept `error` rule does not apply to declared snapshot assets.
- **Unknown `type` value**: tolerated, never a finding (OKF §4.1 — consumers must tolerate unknown types).
- **Unknown / extension frontmatter keys**: preserved on round-trip, never rejected.
- **Broken cross-link**: not a finding — OKF §5.3 requires consumers to tolerate links to not-yet-written knowledge. Inbound-link resolution and referential integrity are C005's responsibility, not C004's.
- **Reserved filename used as a concept** (`index.md` / `log.md` holding concept content): `error`.
- **Slug collision** when deriving a filename: the slug helper disambiguates deterministically (e.g. a numeric suffix) rather than returning a path that would overwrite an existing concept.
- **`okf_version` in the root `index.md` newer than the pinned version**: `advisory`, surfaced for the steward (best-effort consumption per OKF §11), not a hard failure.

## Relationships

- **C003 - Integration Authoring**: primary consumer of the write face — serializes frontmatter, derives slugs/paths, formats citation blocks and cross-links, then writes the concept directly; may call `validate` on the result.
- **C005 - Index & Navigation**: uses the index-rendering and link-syntax conventions; C004 validates `index.md` *structure*, but link *integrity* (inbound resolution, dangling references) is owned by C005.
- **C007 - Currency Tracking**: uses the `log.md` entry format and the `timestamp` field convention to record recency and history; C004 checks the shape/presence of `timestamp`, while writing and maintaining its value is C007's.
- **C008 - Lifecycle & Retirement**: uses the deprecation-marker convention to mark concepts deprecated or superseded.
- **C002 - Triage** (and any concept reader): uses the parse helper to read frontmatter and body without encoding the format itself.
- **Boundary**: C004 owns format *shape* only. It does not own semantic content rules — whether every claim is cited (C003 authors, C006 revalidates), whether a concept is in scope (C002 / C010), or whether cross-links resolve (C005). Advisories flag structure that supports those requirements; they never adjudicate the requirement.

## Success criteria

- **One-owner invariant (grep-able)**: no component other than C004 contains OKF structural literals — frontmatter field names, reserved filenames, the bundle-relative link form, or `index.md` / `log.md` structural rules. This is the concrete test of the "one owner of format knowledge" principle.
- **Faithful OKF classification**: `validate` marks every OKF §9 violation as `error` and never emits an `error` for a condition OKF requires consumers to tolerate (unknown `type`, unknown keys, missing optional fields, broken cross-links).
- **Lossless round-trip**: for any conformant concept, `parse → serialize` preserves body bytes and all frontmatter keys, with deterministic key ordering.
- **Authoring floor**: serialized frontmatter always carries a non-empty `type`; the serializer refuses to emit a concept without one.
- **Single point of version change**: a major OKF version bump is implementable by editing only C004 and the vendored pin — verified by the one-owner invariant above.
- **Side-effect-free validation**: `validate` never writes to the corpus.

## Notes

- Pinned to OKF 0.1 at [`vendor/okf/SPEC.md`](../../../vendor/okf/SPEC.md); the pin (upstream commit, hashes, vendored date) is recorded in [`vendor/okf/PROVENANCE.md`](../../../vendor/okf/PROVENANCE.md). Do not edit the vendored spec; re-vendor to change the pin.
- The lowercase-kebab-case slug convention is a **Wiki choice**, not an OKF rule (OKF §3 leaves naming to producers). It is recorded here rather than in a separate decision record; promote it to an ADR if it becomes contested.
- The product's **Lint** operation (engineering README, Technology Choices) is this validator surfaced through the curation Agent Skill.

## See Also

### Architectural Decision Records

- [ADR001 - Single OKF conformance boundary](../drs/ADR001-single-okf-conformance-boundary.md)

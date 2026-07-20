# C003 — OKF Conformance

The single owner of external-format knowledge: normalize content to the open
knowledge format's conventions, and validate content against the pinned spec.
Stateless, deep by behavior — Wiki *consumes* the format, it does not define it.

## Capabilities

C003 is the one place the external Open Knowledge Format (OKF) is understood. It
exposes exactly three calls (frozen — see [INTERFACES.md](../INTERFACES.md) § C003):

- **`apply(content, kind) -> ConformantContent`** — turn draft content of a given
  `kind` into content that satisfies the format's structural + naming conventions,
  so a contributor never has to know them (O004-R002).
- **`validate(target) -> ConformanceReport`** — judge whether a target already
  conforms to the pinned spec, reporting `ok` or the specific violations (O006-R002).
- **`spec_version() -> Version`** — report which pinned OKF version this component
  currently embeds.

No other capability leaks format knowledge out of this boundary.

## Data model

**Stateless — owns no persistent artifact.** C003 holds no corpus state and no log;
it is pure with respect to the corpus. What it *embeds* (in code, not in the corpus)
is two things:

- the **pinned OKF spec version** — a single explicit `Version` identifier plus the
  vendored spec snapshot it names; and
- the **mapping** from OKF rules to the concrete structural + naming conventions Wiki
  applies and checks (headings/sections, file/path naming, required fields, link form).

Shared types (see [DECOMPOSITION.md](../DECOMPOSITION.md#shared-types)):
`ConformanceReport = ok | [violation]`, `Version`. `ConformantContent` is the
apply-normalized content C008 then writes.

## Behavior

**`apply`** — normalize `content` of a declared `kind` to the format's structural and
naming conventions and return `ConformantContent`. Called by **C008 only**, the sole
corpus writer; C003 never writes the corpus itself. `apply` is the "hands" half of
convention encapsulation — the contributor supplies intent, C003 supplies conformance.

**`validate`** — check a `target` against the **pinned** spec and return `ok` or the
list of violations. Two caller contexts, one behavior: C008 calls it **pre-write** (a
write that would not validate does not commit), and C009 calls it for **audit** of
already-written corpus (classifying dangling targets as malformed vs. merely absent). `validate` is read-only and side-effect-free; it neither
repairs nor records — remediation is a separate act driven through C008.

**`spec_version`** — return the embedded pinned `Version`, so callers and provenance
can state exactly which format revision a judgment was made against.

**Deep by behavior, not by data.** All three calls are narrow; the depth is the format
knowledge behind them, not any state they carry.

### Upgrade handling

The pinned version is bumped by a **deliberate, one-component change**: editing the
embedded version identifier + vendored snapshot + rule mapping in C003, and nowhere
else. C003 does **not** auto-track upstream OKF releases — the format is an early,
single-origin draft (O006's semantic-portability layer), so adoption of a new revision
is a reviewed engineering decision, not a floating dependency.

Because C003 is stateless, `validate` always judges against the *currently* pinned
spec. On a bump, already-conformant content is **not** silently migrated and is never
destroyed: content authored under the old revision may now surface fresh violations on
its next `validate`. Those violations are ordinary conformance signals — remediated by
re-running `apply` through C008 on the normal write path (e.g. a lifecycle-driven
re-conformance sweep). The corpus stays plain-text and readable throughout; only its
degree of conformance to the *new* revision changes, and it converges as content is
rewritten. This is precisely the one-component blast radius the architecture promises.

## Relationships

- **Depends on nothing** below it — stateless, embeds its own format knowledge.
- **Called by C008** (`apply` + `validate`, on every write) and by **C009**
  (`validate`, for audit). Matches [INTERFACES.md](../INTERFACES.md) dependency
  direction exactly: `C003 ◀── C008, C009`.
- It is the **only** component that embeds format rules. No caller re-implements or
  inspects OKF conventions; they express intent (C008) or request a report
  (C009). C003 owns the **semantic-portability** layer (open-format conformance);
  the **substrate-portability** layer (plain-text + version control) is a property of
  how C008/C006 store the corpus, not of this component.

## Decisions

**Single conformance owner (settles old ADR001).** The boundary is re-derived directly
from two product requirements: convention encapsulation (O004-R002 — apply conventions
automatically so no contributor needs to know them) and open-format conformance
(O006-R002 — keep the corpus conformant to an *externally* published, versioned format
Wiki consumes rather than defines). Together they demand exactly one place that holds
format knowledge, so a format-version bump is a one-component change. This is a textbook
deep-module boundary; per the [DECOMPOSITION.md](../DECOMPOSITION.md) old-ADR checklist
it needs **no separate ADR** — it is documented here.

**Internal choice — how the spec is pinned (resolved).** Pin to an *explicit* version
identifier plus a vendored spec snapshot embedded in the component, never a floating
"latest". Rationale: deterministic validation (the same target validates the same way
until a deliberate bump) and a consumes-not-defines posture toward an early external
draft. No cross-component fork — internal to C003.

## Success criteria

Tied to the O006 proxy — *proportion of the corpus that is plain-text,
version-controlled, and validates against the pinned open-format spec*:

- Every write goes through `apply`, and `validate` gates it, so newly authored content
  is conformant by construction; the O006 proxy trends up as a direct result.
- `validate` gives C009 an honest, current conformance signal over existing corpus
  (including newly-surfaced violations after a bump), so gaps are visible, not hidden.
- A format-version bump is realized by editing **C003 only**; no other component
  changes, and the corpus remains plain-text and readable throughout the transition.

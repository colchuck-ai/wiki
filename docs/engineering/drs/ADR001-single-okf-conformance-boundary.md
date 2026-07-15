# ADR001 - Single OKF conformance boundary

Confine all knowledge of the OKF format — the authoring conventions the tool writes to and the validator that checks the corpus — to one component, C004 - OKF Conformance, so that no other component or contributor encodes the format and an OKF version change is absorbed at a single boundary.

## Context

Wiki does not define or version the knowledge format; it consumes a published, versioned OKF (a product non-goal), pinned in-repo at `vendor/okf/SPEC.md` (OKF 0.1). Nearly every component reads or writes concepts — C001 through C010 all touch OKF files following its conventions — and the architecture requires them to do so directly on the git-tracked substrate, without shared mutable state.

Two pressures follow. First, OKF is an external, evolving spec: its own §11 anticipates minor bumps (new optional fields, new conventional headings) and major bumps (renamed required fields, changed reserved filenames). Second, if each component embeds its own understanding of frontmatter fields, reserved filenames, link syntax, and index/log structure, that knowledge scatters across the system — a format change then ripples through every component, and conformance can drift component-by-component with nothing that authoritatively defines "conformant."

## Options

- **Single conformance boundary (chosen)**: one component (C004) owns the authoring conventions and the validator; all other components produce and check OKF output through its helpers while doing their own reads and writes. Format knowledge lives in one place; the version pin lives in one place.
- **Shared format library, no owning component**: expose serialization/parse helpers as a passive utility every component imports, but with no single component accountable for conformance or the validator. Removes duplication but diffuses ownership — there is no one place the "is the corpus conformant?" question resolves, and version pinning has no home element.
- **Per-component OKF handling**: each component reads and writes OKF directly with its own inline understanding of the format. Simplest per component in isolation, but scatters format knowledge, invites drift between components, and turns every OKF bump into a system-wide edit.

## Decision

Adopt the single conformance boundary. C004 - OKF Conformance is the sole owner of format knowledge, exposing authoring conventions (frontmatter serialization, slug/path derivation, index/log/citation/link formatting, deprecation marker) and a two-severity validator (OKF §9 errors plus a Wiki structural-profile advisory tier). Other components call these helpers and write concept files themselves, keeping the architecture's "no shared mutable state" while ensuring the format is defined once. C004 binds to exactly one pinned OKF version and is the boundary at which version changes are absorbed. This realizes the architecture's "one owner of format knowledge" principle as a concrete component contract.

## Consequences

- Format knowledge is defined once; the "one-owner invariant" (no OKF structural literal appears outside C004) is a grep-able check enforcing this over time.
- An OKF version change is localized: a minor bump adds tolerated fields/headings in C004; a major bump is a deliberate C004 change against the re-vendored pin, with no edit to any other component. The pin and re-vendoring procedure live in `vendor/okf/PROVENANCE.md`, which points here.
- C004 becomes a shared dependency on effectively every concept read and write path; a defect in it affects all components, so its correctness and round-trip fidelity carry outsized weight (reflected in C004's success criteria).
- C004's interface must be broad enough to cover every consumer's authoring need (C003, C005, C007, C008), creating pressure toward a wide helper surface that must be kept cohesive rather than becoming a catch-all.
- Contributors and agents need not learn OKF to add conformant knowledge, directly serving convention encapsulation (O004-R002).

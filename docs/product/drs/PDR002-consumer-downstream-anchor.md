# PDR002 - Consumer job as a downstream anchor

We decided to model Wiki and its output as **two products with two hirers** —
the steward hires *Wiki* (the tool); the consumer hires *the body of knowledge*
(the deliverable) — and to document the consumer's job as an explicit
**downstream anchor** that the deliverable-quality outcomes trace to, rather
than as a second job measured on Wiki or as needs left implicit inside the
steward's frame.

This decision affects the product statement (`docs/product/README.md`): it adds
an Actors section, a "deliverable and its downstream job" section, and an
outcome-family tag on every outcome.

## Context

The spine sentence has always served two populations — the steward who *hires
Wiki* and the consumer who "can find and rely on what we know." But the tree
framed everything as a single job owned by the steward, so outcomes that are
realized in the *consumer's* moment of use (O001 "at the moment they rely on
them," O005 "time to locate," O009-R002 "so a consumer can tell") were written
from the steward's frame with the consumer left nameless. That obscured *who
each outcome is measured for* and let consumer-serving outcomes hide inside a
steward job.

The clarifying observation: the consumer never touches Wiki. They hire a
*different* product — the body of knowledge Wiki outputs. This is the
tool-produces-artifact pattern (a CMS is hired by editors; the site it outputs
is hired by readers).

## Options

- **Leave the consumer implicit** (status quo): consumer needs stay smuggled
  into outcome wording; "who is this outcome for" stays ambiguous.
- **Add a full second job measured on Wiki**: names the consumer, but makes Wiki
  accountable for an outcome it does not control end-to-end — the consumer's
  success also depends on their own question, skill, and reader.
- **Document the consumer's job as a downstream anchor** (chosen): a named job
  the *deliverable* serves, from which the deliverable-quality outcomes derive
  their existence and metrics, while Wiki keeps its single *measured* job.

## Decision

Model two products and two hires. Keep **Knowledge Stewardship** as Wiki's sole
measured job. Document the **consumer's downstream job** as an anchor, not a
measured job. Split the outcome set into two families and tag each outcome:

- **deliverable-quality** (owed to the consumer): O001, O002, O003, O005, O006,
  O007, O008, O009.
- **curation-experience** (the steward's own experience): O004 — the lone
  outcome that would still matter if no consumer ever existed, which is the tell
  that the split is real.

The consumer's job is not measured on Wiki because Wiki controls the corpus,
not the consumer's session.

## Consequences

- Every deliverable-quality outcome now has an honest *source*: its metric is
  read against the downstream job, which is why each also carries an observable
  **proxy** (see [PDR003](PDR003-manage-by-exception.md) neighbors and the
  Operating principles section).
- The spine sentence is decomposed honestly across all deliverable properties
  (traceable, faithful, current, high-signal, discoverable, on-scope, complete,
  portable) plus the steward's low-effort — closing the old gap where
  "trustworthy" named five properties while the tree had nine outcomes.
- The consumer is explicitly *a teammate or agent*, which sets up the
  machine-readable-first principle: everything human-legible must also be
  machine-readable, because half the declared consumers are agents.

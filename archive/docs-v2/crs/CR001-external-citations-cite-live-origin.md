# CR001 - External citations cite the live origin

Updated C003 - Integration Authoring's citation behavior for captured-snapshot sources: the citation now links the live `origin` URL instead of the archived `/references/` path.

## Change

Before: for a captured-snapshot source, C003 cited the promoted `/references/` asset by its bundle-relative path — the citation pointed only at the durable local copy, never at the live external locator.

After: for a captured-snapshot source, C003 cites the live `origin` URL as the citation's primary link (an ordinary external citation per OKF §8), and names the archived `/references/` asset as a durable fallback copy in the surrounding prose. An in-place source's citation is unchanged — it still cites its existing repo path. C003's document (Data model → Citation, Behavior → promotion/citation, Relationships → C006) is updated accordingly.

## Rationale

Specifying C006 - Source Revalidation surfaced a gap: nothing in the corpus recorded the live locator a captured-snapshot claim needs to be re-checked against, since the citation pointed only at the immutable archived copy. Reaching back into C001 - Ingestion Queue's task record for `origin` would make C001's staging queue a permanent ledger it was never designed to be. Citing the live origin directly keeps C006 self-contained — it reads what it needs straight off the concept through C004, with no dependency on C001 after authoring. See [ADR007](../engineering/drs/ADR007-citation-link-form-for-drift-revalidation.md) for the full option analysis and the on-demand-baseline decision this change enables.

## Affects

- **C003 - Integration Authoring** — the modified element; its Data model, Behavior, and Relationships sections are updated.
- **C006 - Source Revalidation** — the new component this change unblocks; specced against the resulting citation contract.
- **ADR007** — records the decision and alternatives considered.

# Vendored: Open Knowledge Format (OKF)

`SPEC.md` in this directory is a **pinned, byte-identical copy** of the Open
Knowledge Format specification. Wiki does not define or own this format — it
consumes a published, versioned OKF (a product non-goal; see the engineering
README and `ADR001-single-okf-conformance-boundary.md`). This copy is vendored
so the spec Wiki builds against is present in the repo rather than only referred
to by an external URL.

Do not edit `SPEC.md` here. A format change is an OKF change, not a Wiki change:
update the pinned copy by re-vendoring from upstream and bumping the record
below.

## Pin

| Field           | Value                                                              |
|-----------------|--------------------------------------------------------------------|
| Format version  | OKF 0.1 (Draft)                                                    |
| Upstream repo   | https://github.com/GoogleCloudPlatform/knowledge-catalog           |
| Upstream path   | `okf/SPEC.md`                                                      |
| Upstream commit | `ee67a5ca27044ebe7c38385f5b6cffc2305a9c1a` (2026-06-11)           |
| Git blob hash   | `55d0a46cc988e99aa35cd027964d6278a4f93f35`                         |
| SHA-256         | `b9655e607346dbbdc6de21190e9a953313eda6a7eba68d4d272a65975940ad6e` |
| Vendored on     | 2026-07-14                                                         |

Verify the copy is unmodified:

```
shasum -a 256 vendor/okf/SPEC.md
# → b9655e607346dbbdc6de21190e9a953313eda6a7eba68d4d272a65975940ad6e
```

## Re-vendoring

```
cp <upstream>/okf/SPEC.md vendor/okf/SPEC.md
```

Then update the Format version, Upstream commit, Git blob hash, SHA-256, and
Vendored-on rows above. A **major** OKF bump may change required fields or
reserved filenames (see SPEC §11) and is absorbed at C004 - OKF Conformance
Adapter per ADR001 — treat it as a deliberate Adapter change, not a drop-in.

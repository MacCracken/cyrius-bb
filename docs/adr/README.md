# Architecture Decision Records

Decisions about cyrius-bb.

## Conventions

- **Filename**: `NNNN-kebab-case-title.md`, zero-padded to four digits. Never renumber.
- **One decision per ADR.** Supersessions add a new ADR and mark the old `Superseded by NNNN`.
- **Status lifecycle**: `Proposed` → `Accepted` → (optionally) `Superseded` or `Deprecated`.
- Use [`template.md`](template.md) to start a new ADR.

## Index

- [0001 — Homage from observation, not source port](0001-homage-from-observation.md) — mechanics traced to documented public sources; no reverse-engineering of original Atari ROM / binary.
- [0002 — Original assets only](0002-original-assets-only.md) — new art / audio / level data or public-domain reference material; no Atari-era assets shipped.

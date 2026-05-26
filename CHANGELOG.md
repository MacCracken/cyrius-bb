# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed
- Toolchain pin bumped `5.7.11` → `6.0.1` in `cyrius.cyml` (latest Cyrius). All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.
- Repointed stale first-party-standards links `docs/development/applications/` → `docs/development/first-party/` in `CLAUDE.md` and `docs/development/roadmap.md`; added shared-crates + doc-health pointers to `CLAUDE.md`.

### Added
- `src/fixed.cyr` — deterministic 16.16 fixed-point math primitives (`asr`, `fixed_mul/div/clamp/abs`, int conversions). Self-contained: `asr` defined locally since cyrius-bb does not vendor bsp.
- `src/geom.cyr` — collision + reflection primitives: `aabb_overlap`, `reflect` (axis-aligned bounce), and `paddle_english` / `paddle_bounce_vx` (the documented Breakout outbound-angle mechanic, ADR 0001).
- `tests/cyrius-bb.tcyr` — 17 new assertions across `fixed` + `geom` groups (19 total with smoke). All green.
- [ADR 0003](docs/adr/0003-self-rolled-primitives.md) — defer kiran / impetus / mabda / soorat; tool primitives on bare stdlib (cyrius-doom pattern). sankoch + sigil retained for the M5 save file.
- `docs/doc-health.md` — whole-tree doc-currency ledger (fresh / stale / archive / open-question), adapted from the cyrius reference and scaled to the small tree. Created ahead of the ~30-doc threshold by request.
- Required root files `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` — adapted from sibling first-party repos; complete the required-root set. CONTRIBUTING leads with the ADR 0001/0002 hard constraints and the playtest gate; SECURITY names the save file as the only untrusted-input surface.

## [0.1.0]

### Added
- Initial project scaffold

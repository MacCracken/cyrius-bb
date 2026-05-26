# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.1.0] — 2026-05-25

Initial release: scaffold, deterministic game primitives, and full first-party doc compliance. No gameplay loop yet — the build approach is captured in [ADR 0003](docs/adr/0003-self-rolled-primitives.md). DCE release binary: **748,032 bytes** (x86_64, static, stripped).

### Added
- Project scaffold (`cyrius init`): `src/main.cyr`, test/bench/fuzz harnesses, CI + release workflows, docs tree, [ADR 0001](docs/adr/0001-homage-from-observation.md) (homage-from-observation) + [ADR 0002](docs/adr/0002-original-assets-only.md) (original-assets-only).
- `src/fixed.cyr` — deterministic 16.16 fixed-point math primitives (`asr`, `fixed_mul/div/clamp/abs`, int conversions). Self-contained: `asr` defined locally since cyrius-bb does not vendor bsp.
- `src/geom.cyr` — collision + reflection primitives: `aabb_overlap`, `reflect` (axis-aligned bounce), and `paddle_english` / `paddle_bounce_vx` (the documented Breakout outbound-angle mechanic, ADR 0001).
- `tests/cyrius-bb.tcyr` — 19 assertions across `smoke` + `fixed` + `geom` groups. All green; lint + fmt clean.
- [ADR 0003](docs/adr/0003-self-rolled-primitives.md) — defer kiran / impetus / mabda / soorat; tool primitives on bare stdlib (cyrius-doom pattern). sankoch + sigil retained for the M5 save file.
- `docs/doc-health.md` — whole-tree doc-currency ledger (fresh / stale / archive / open-question), adapted from the cyrius reference and scaled to the small tree.
- Required root files `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` — adapted from sibling first-party repos; complete the required-root set. CONTRIBUTING leads with the ADR 0001/0002 hard constraints and the playtest gate; SECURITY names the save file as the only untrusted-input surface.

### Changed
- Toolchain pinned to Cyrius **6.0.1** (from the `5.7.11` scaffold pin). All 16 stdlib modules in the dep list resolve against 6.0.1.
- Repointed first-party-standards links `docs/development/applications/` → `docs/development/first-party/` (`CLAUDE.md`, `roadmap.md`); added shared-crates + doc-health pointers to `CLAUDE.md`.
- CI/release workflows reworked to match the cyrius-doom reference: version-pinned toolchain install (`~/.cyrius/versions/$V/`, which the stdlib resolver prefers) + HTTP pre-flight, `cyrius.lock` presence gate, guarded `cyrius deps --verify` (around the 6.0.1 empty-lock regression), a `docs` job (required-files + version consistency), and `CYRIUS_DCE=1` on the release build + SHA256SUMS.

### Fixed
- CI was unrunnable as scaffolded: `ci.yml` lacked the `workflow_call:` trigger that `release.yml`'s CI-gate depends on, and the test step pointed at a non-existent `src/test.cyr` (tests live in `tests/cyrius-bb.tcyr`). Both fixed.

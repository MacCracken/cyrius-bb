# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed
- Toolchain pin bumped `5.7.11` → `6.0.1` in `cyrius.cyml` (latest Cyrius). All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.
- Repointed stale first-party-standards links `docs/development/applications/` → `docs/development/first-party/` in `CLAUDE.md` and `docs/development/roadmap.md`; added shared-crates + doc-health pointers to `CLAUDE.md`.

### Added
- `docs/doc-health.md` — whole-tree doc-currency ledger (fresh / stale / archive / open-question), adapted from the cyrius reference and scaled to the small tree. Created ahead of the ~30-doc threshold by request.
- Required root files `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` — adapted from sibling first-party repos; complete the required-root set. CONTRIBUTING leads with the ADR 0001/0002 hard constraints and the playtest gate; SECURITY names the save file as the only untrusted-input surface.

## [0.1.0]

### Added
- Initial project scaffold

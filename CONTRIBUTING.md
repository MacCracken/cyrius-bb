# Contributing to cyrius-bb

Thank you for your interest in contributing to **cyrius-bb** (*break-breaker*) —
a 2.5D brick-breaker in Cyrius, homage to *Breakout* (Atari, 1976).

Read [`CLAUDE.md`](CLAUDE.md) first — it is the durable process and procedures
for this repo. This file is the contributor-facing summary.

## Two hard constraints, before anything else

cyrius-bb is a *homage built from observation*, not a port. Two ADRs govern
every contribution and are non-negotiable:

- **Original assets only** ([ADR 0002](docs/adr/0002-original-assets-only.md)) —
  no Atari-era sprites, sounds, or level data. All art, audio, and level design
  is newly created or drawn from public-domain reference material. No ripping,
  no "inspired-by" substitutes pretending to be the original.
- **Homage from observation** ([ADR 0001](docs/adr/0001-homage-from-observation.md)) —
  mechanics trace to documented public sources (interviews, design writeups,
  gameplay footage). Do **not** reverse-engineer the original ROM / binary,
  disassemble it, or feed it to an LLM. The sovereignty stance includes the
  build process.

A PR that ships Atari-era assets or derives mechanics from the original binary
will be declined regardless of code quality.

## Prerequisites

- [Cyrius](https://github.com/MacCracken/cyrius) at the version pinned in
  [`cyrius.cyml`](cyrius.cyml) `[package].cyrius` (currently `6.0.1`) — ships
  the `cyrius` CLI and the stdlib cyrius-bb depends on.
- No C toolchain, no FFI. The Cyrius game stack (mabda / kiran / impetus /
  shravan) and stdlib carry everything — see [`README.md`](README.md).

Everything (deps, build, test, lint, fmt) runs through the `cyrius` CLI.

## Development Workflow

1. Fork and clone the repository
2. Resolve deps: `cyrius deps`
3. Make changes under `src/`, `tests/`, or `docs/`
4. Build: `cyrius build src/main.cyr build/cyrius-bb`
5. Unit tests: `cyrius test src/test.cyr`
6. **Playtest** — run the built binary. For ball physics, paddle english, and
   destruction rhythm, the *run* is the gate, not the unit test. A green test
   suite does not prove the game feels like Breakout (see *Playtest over unit
   test* below).
7. Lint clean: `cyrius lint src/*.cyr`
8. Submit a PR

## Project Structure

cyrius-bb is at scaffold stage (v0.1.0 — no gameplay code yet). The planned
module layout (tracked in [`docs/development/state.md`](docs/development/state.md)):

```
src/main.cyr      entry, game loop, scene switching
src/ball.cyr      ball physics + render
src/paddle.cyr    paddle input + render + english
src/bricks.cyr    brick layout + destruction + scoring
src/level.cyr     level loading, progression, speed-curve
src/hud.cyr       score, lives, level indicator
src/audio.cyr     sound-effect synthesis via shravan
src/save.cyr      high-score file I/O (sankoch + sigil)
src/test.cyr      unit-test entrypoint
```

See [`docs/development/roadmap.md`](docs/development/roadmap.md) for the
milestone sequence (M0 scaffold → M1 ball/paddle/bricks → … → v1.0).

## Playtest over unit test (for feel)

Unit tests prove **determinism and boundary conditions** — collision math,
buffer bounds, save-file roundtrips. Playtest proves the game is **fun**. If a
change touches ball physics, paddle english, or the speed curve, the PR must
say how it played, not just that the tests pass. Feel observations go in the
CHANGELOG **Feel** subsection.

## Code Style

Follow the Cyrius conventions in [`CLAUDE.md`](CLAUDE.md):

- `var buf[N]` is N **bytes**, not N elements
- `&&` / `||` short-circuit; mixed conditions require parens
- No closures — named functions
- No negative literals — write `(0 - N)`, not `-N`
- Test exit pattern: `syscall(60, assert_summary())`
- Use `#` comments
- `cyrius fmt` is the arbiter of formatting; CI fails on drift

## Adding a Module

1. Create `src/mymodule.cyr`
2. `include` it from `src/main.cyr` in dependency order
3. Add unit tests reached from `src/test.cyr` (give each a `test_` prefix)
4. Build + test + playtest the path the module touches
5. Update [`CHANGELOG.md`](CHANGELOG.md) and, if a milestone boundary is
   crossed, [`docs/development/state.md`](docs/development/state.md)

## Commit Messages & CHANGELOG

Follow [Keep a Changelog](https://keepachangelog.com/) categories:

- `add: feature description` — new gameplay or systems
- `fix: bug description` — bug fixes
- `change: what changed` — modifications
- `feel: observation` — playtest-driven tuning (speed curve, english, rhythm)

Gameplay-mechanic changes are the primary CHANGELOG entry type; playtest notes
go in a **Feel** subsection.

## Security

Found a vulnerability? See [`SECURITY.md`](SECURITY.md) — report via GitHub
Security Advisories, not a public issue.

## License

By contributing, you agree that your contributions are licensed under
**GPL-3.0-only**.

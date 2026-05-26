# cyrius-bb — Claude Code Instructions

> **Core rule**: this file is **preferences, process, and procedures** —
> durable rules that change rarely. Volatile state (current version,
> milestone status, module line counts, binary size, asset inventory,
> high-score-format version) lives in [`docs/development/state.md`](docs/development/state.md).
> Do not inline state here.

## Project Identity

**cyrius-bb** (*break-breaker*) — 2.5D brick-breaker in Cyrius. Homage to *Breakout* (Atari, 1976).

- **Type**: Binary (game)
- **License**: GPL-3.0-only
- **Language**: Cyrius (toolchain pinned in `cyrius.cyml [package].cyrius`)
- **Version**: `VERSION` at the project root is the source of truth
- **Genesis repo**: [agnosticos](https://github.com/MacCracken/agnosticos)
- **Standards**: [First-Party Standards](https://github.com/MacCracken/agnosticos/blob/main/docs/development/first-party/first-party-standards.md) · [First-Party Documentation](https://github.com/MacCracken/agnosticos/blob/main/docs/development/first-party/first-party-documentation.md)
- **Shared crates**: [shared-crates.md](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/shared-crates.md)

## Goal

Ship a structurally-faithful brick-breaker in Cyrius. Deliberately the simplest technical bet in the retro-port-in-Cyrius series — the point is **stack demonstration at accessible scope**, not engineering-novelty on the scale of cyrius-braid's time-rewind buffer. Ball, paddle, bricks, 2.5D depth layering, original sound + art, public high-score table via sigil-hashed + sankoch-compressed save file. That's the whole game.

The homage surface: structural fidelity to Breakout's documented mechanics (wall-bounce angles, paddle-english physics, brick-destruction rules, progressive speed increase across levels). Captured in [ADR 0001](docs/adr/0001-homage-from-observation.md).

## Current State

> Volatile state — current version, milestone status, binary size,
> asset count, active level count, high-score-format version —
> lives in [`docs/development/state.md`](docs/development/state.md).
> Refreshed every release.

This file (`CLAUDE.md`) is durable rules.

## Scaffolding

Project was scaffolded with `cyrius init cyrius-bb` on 2026-04-24. **Do not manually create project structure** — use the tools. If the tools are missing something, fix the tools.

## Quick Start

```bash
cyrius deps                                                # resolve stdlib deps
cyrius build src/main.cyr build/cyrius-bb                  # build
cyrius test src/test.cyr                                   # unit tests
CYRIUS_DCE=1 cyrius build src/main.cyr build/cyrius-bb     # release-parity build
```

## Key Constraints

- **Reimplementation from observation, not source port.** Atari Breakout's source was never publicly released. Every mechanic traces to documented public sources (interviews, design writeups, gameplay footage); the original binary is never reverse-engineered. See [ADR 0001](docs/adr/0001-homage-from-observation.md).
- **Original assets only.** No Atari-era sprites / sounds / level data. All art, audio, and level design is newly created or drawn from public-domain reference material. See [ADR 0002](docs/adr/0002-original-assets-only.md).
- **2.5D is visual treatment, not engine complexity.** Orthogonal 2D with depth layering (parallax, brick-destruction perspective); not a 3D engine. If a design choice requires full 3D, it's out of scope — that's a different project.
- **Playtest over unit test for feel.** Ball physics, paddle english, and the destruction-rhythm that makes Breakout *feel* like Breakout are playtest concerns. Unit tests prove determinism and boundary conditions; playtest proves the game is fun.
- **No ML-assisted mechanics-reverse-engineering.** The rebuild works from documented public sources. Don't feed the original ROM / binary to anything; don't disassemble; don't LLM-read the code. Sovereignty stance includes the build process.
- **No FFI.** Same rule as the rest of AGNOS.

## Development Process

### Work Loop

1. Pick the next item from `docs/development/roadmap.md`
2. Implement against the Cyrius game stack (mabda / kiran / impetus / shravan)
3. `cyrius build` → `cyrius test` → run the built binary
4. Playtest — if the mechanic matters, the run matters. A unit test alone doesn't prove the feel is right
5. Update `CHANGELOG.md` and `docs/development/state.md` if milestone boundary crossed
6. Version bump only at milestone close

### Closeout Pass (before minor/major bump)

1. Full test suite — all `.tcyr` green
2. Manual playthrough of all shipped levels, end to end
3. `cyrius lint src/*.cyr` — clean
4. `CYRIUS_DCE=1 cyrius build` — release binary; note size in CHANGELOG + `state.md`
5. Version triple (`VERSION`, `cyrius.cyml`, CHANGELOG header) in sync
6. `state.md` current — level count, asset count, binary size, milestone status

## Key Principles

- **Accessible scope** — cyrius-bb is deliberately simple. Don't let scope creep in; the point is the stack demo, not an engineering-novelty showcase
- **Feel is playtest territory** — ball physics need to feel right to a human, not just pass unit tests
- **Reference the documented original; don't reverse-engineer** — sovereignty stance applies to how we build, not just what we ship
- **Original assets in the spirit of the era** — no ripping, no passing-for-original substitutes

## Rules (Hard Constraints)

- **Read the genesis repo's CLAUDE.md first** — [agnosticos/CLAUDE.md](https://github.com/MacCracken/agnosticos/blob/main/CLAUDE.md)
- **Do not commit or push** — the user handles all git operations
- **NEVER use `gh` CLI** — use `curl` to the GitHub API if needed
- Do not ship Atari-era assets. Original art / audio / level data only
- Do not reverse-engineer the original ROM / binary — documented mechanics only
- Do not add FFI — mabda / kiran / impetus / shravan carry everything
- Do not inline volatile state in this file — `docs/development/state.md` is the home

## Cyrius Conventions

- `var buf[N]` is N **bytes**, not N elements
- `&&` / `||` short-circuit; mixed requires parens
- No closures — named functions
- No negative literals — `(0 - N)`, not `-N`
- Test exit pattern: `syscall(60, assert_summary())`

## Docs

- [`docs/adr/`](docs/adr/) — architecture decision records. Start with ADR 0001 (homage-from-observation) + ADR 0002 (original-assets-only).
- [`docs/architecture/`](docs/architecture/) — non-obvious constraints and quirks
- [`docs/guides/`](docs/guides/) — task-oriented how-tos
- [`docs/examples/`](docs/examples/) — runnable examples
- [`docs/design/`](docs/design/) — art direction, asset pipeline, palette / audio sourcing
- [`docs/development/roadmap.md`](docs/development/roadmap.md) — milestone sequence (M0 scaffold → M1 ball/paddle/bricks → M2 levels → M3 2.5D depth pass → M4 audio + polish → v1.0)
- [`docs/development/state.md`](docs/development/state.md) — live state snapshot
- [`docs/doc-health.md`](docs/doc-health.md) — fresh / stale / archive ledger across the whole doc tree (lives at `docs/` root, **not** under `development/` — its scope is the whole tree)
- [`CHANGELOG.md`](CHANGELOG.md) — source of truth for all changes

New quirks and constraints land in `docs/architecture/` as numbered items (`NNN-kebab-case.md`). New decisions land in `docs/adr/` using [`template.md`](docs/adr/template.md). **Never renumber either series.**

## CHANGELOG Format

Follow [Keep a Changelog](https://keepachangelog.com/). Gameplay-mechanic changes are the primary entry type; playtest-observation notes (feel tweaks, speed-curve adjustments) get a **Feel** subsection.

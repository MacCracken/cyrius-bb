# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.1.0** — scaffolded 2026-04-24 via `cyrius init cyrius-bb`. Module skeletons + ADR 0001 (homage-from-observation) + ADR 0002 (original-assets-only) landed first. No gameplay code yet.

## Toolchain

- **Cyrius pin**: `6.0.1` (in `cyrius.cyml [package].cyrius`) — bumped from `5.7.11` 2026-05-25. All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + single brick layer, wall/paddle/brick collision, basic render loop
- M2 — level system (5+ levels, increasing speed / brick count)
- M3 — 2.5D depth pass (parallax background, brick-destruction perspective, particle debris)
- M4 — audio pass (shravan-synthesized effects, optional slot-loaded music)
- M5 — high-score file (sankoch-compressed, sigil-integrity-hashed)
- v1.0 — end-to-end playable, polish complete

## Source

Scaffold only.

Planned modules:
- `src/main.cyr` — entry, game loop, scene switching
- `src/ball.cyr` — ball physics + render
- `src/paddle.cyr` — paddle input + render + english
- `src/bricks.cyr` — brick layout + destruction + scoring
- `src/level.cyr` — level loading, progression, speed-curve
- `src/hud.cyr` — score, lives, level indicator
- `src/audio.cyr` — sound effect synthesis via shravan
- `src/save.cyr` — high-score file I/O (sankoch + sigil)

## Assets

Scaffold — no assets shipped yet.

Planned per [ADR 0002](../adr/0002-original-assets-only.md):
- Original sprite set (paddle, ball, brick variants) — new pixel art
- Era-spirit palette (not pixel-matching Atari Breakout's specific row colors)
- Original square-wave / FM sound effects via shravan
- Placeholder level layouts during M1–M2 development; intentional layouts by M5

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — unit tests (not yet written)
- Playtest gate — every milestone requires manual playthrough (feel concerns)

## Dependencies

All folded into Cyrius stdlib:
- **Core**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Game stack**: mabda (GPU), kiran (engine, planned), impetus (physics, planned), shravan (audio)
- **Save / scoring**: sankoch (compression), sigil (hash)

## Next

1. **M1** — ball / paddle / bricks foundational loop. Load-bearing technical surface (collision physics + render loop); everything stacks on it.
2. **M2** — levels. Once M1 is tunable, turn the single level into a progression.
3. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone. Outline captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

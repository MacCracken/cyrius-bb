# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.1.0** — cut 2026-05-25. Scaffolded 2026-04-24 via `cyrius init cyrius-bb`; this release adds the first deterministic game primitives (`fixed.cyr`, `geom.cyr`) + full first-party doc compliance. No gameplay loop yet (see [ADR 0003](../adr/0003-self-rolled-primitives.md)).

- **DCE binary**: 748,032 B (x86_64, static, stripped) — still includes unused mabda surface; shrinks after the manifest-cleanup bite.
- **Tests**: 19 assertions (smoke + fixed + geom), 0 failed. Lint + fmt clean.

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

Primitives landed; entity/loop/render/input pending. Per [ADR 0003](../adr/0003-self-rolled-primitives.md), cyrius-bb tools its own primitives on bare stdlib (cyrius-doom pattern) rather than waiting on kiran/impetus/mabda.

Present:
- `src/main.cyr` — entry; currently a mabda link smoke test (to be replaced by the game loop)
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english

Planned modules:
- `src/ball.cyr` — ball entity, velocity integration
- `src/paddle.cyr` — paddle entity + input
- `src/bricks.cyr` — brick grid, destruction, scoring
- `src/world.cyr` — `world_step()` tick: integrate → collide → score
- `src/framebuf.cyr` — `/dev/fb0` framebuffer surface + present (self-rolled)
- `src/input.cyr` — keyboard (self-rolled)
- `src/level.cyr` — level loading, progression, speed-curve
- `src/hud.cyr` — score, lives, level indicator
- `src/audio.cyr` — sound-effect synthesis
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

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input — on bare stdlib
- **Save / scoring**: sankoch (compression), sigil (hash) — retained for M5
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing). Still wired in `cyrius.cyml` as active deps pending a manifest-cleanup bite.

## Next

1. **M1** — ball / paddle / bricks foundational loop. Load-bearing technical surface (collision physics + render loop); everything stacks on it.
2. **M2** — levels. Once M1 is tunable, turn the single level into a progression.
3. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone. Outline captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

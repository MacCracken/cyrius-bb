# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

Last release: **0.1.0** (cut 2026-05-25). Current dev tip (unreleased) adds the full M1 playable loop on self-rolled primitives — see [ADR 0003](../adr/0003-self-rolled-primitives.md) and CHANGELOG `[Unreleased]`.

- **DCE binary**: 114,408 B (x86_64, static, stripped) — down from 748,032 B at 0.1.0 after dropping the dormant mabda/sankoch/sigil deps.
- **Tests**: 84 assertions, 0 failed. Lint + fmt clean.
- **Deps**: bare stdlib — zero external deps (sankoch + sigil re-wire at M5).

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

Present (M1 playable loop — 84 assertions green):
- `src/main.cyr` — real-time loop (tick → input → step → render → present) + headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion
- `src/bricks.cyr` — brick grid + destruction + scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` (re-serve after loss)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()`: bricks/paddle/ball as flat rects
- `src/input.cyr` — keyboard raw-tty input (a/d/arrows/space/q) + pure `bb_key_action` decoder
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit (untested in CI; on-console only)
- `programs/demo.cyr` — eyeball harness; dumps `build/frame00..02.ppm`

Planned modules:
- `src/hud.cyr` — score / lives / level overlay
- `src/level.cyr` — level loading, progression, speed-curve (M2)
- `src/save.cyr` — high-score file I/O (sankoch + sigil) (M5)
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

- `tests/cyrius-bb.tcyr` — **84 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios, framebuf, render, input decode, serve). Deterministic + headless.
- Playtest gate — every milestone requires manual playthrough (feel concerns); pending the render/input/loop bites, the simulation has only the unit-test gate so far.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input — on bare stdlib
- **Save / scoring**: sankoch (compression), sigil (hash) — commented out in `cyrius.cyml`, re-wired at M5
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing) — all commented out in `cyrius.cyml`. The build is now bare-stdlib, warning-free.

## Next

1. **M1** — ball / paddle / bricks foundational loop. Load-bearing technical surface (collision physics + render loop); everything stacks on it.
2. **M2** — levels. Once M1 is tunable, turn the single level into a progression.
3. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone. Outline captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

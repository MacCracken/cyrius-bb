# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.3.0** — M2 (level progression) complete. The single level is now a five-level game: plain-text layout parser, advance-on-clear, per-level speed curve, score + lives carried across levels, game-over on zero lives. Built on M1's self-rolled primitives + bare stdlib ([ADR 0003](../adr/0003-self-rolled-primitives.md)). (Prior: 0.2.0 — M1 foundational loop, 2026-05-25; 0.1.0 — scaffold + primitives, 2026-05-25.)

- **DCE binary**: 103,400 B (x86_64, static, stripped) — up from 98,648 B at 0.2.0 (+4,752 for the level system).
- **Tests**: 127 assertions, 0 failed (was 93). Lint + fmt clean.
- **Deps**: bare stdlib — zero external deps (sankoch + sigil re-wire at M5).
- **Caveat**: interactive loop + `/dev/fb0` present + the 5-level playthrough are build/lint-verified + headless-smoke-verified only (no console/framebuffer in dev/CI); level logic proven via the headless `<frames>` smoke + 127 unit tests.

## Toolchain

- **Cyrius pin**: `6.0.1` (in `cyrius.cyml [package].cyrius`) — bumped from `5.7.11` 2026-05-25. All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5 levels, increasing speed / brick count) (✅ 0.3.0, 2026-05-26)
- M3 — 2.5D depth pass (parallax background, brick-destruction perspective, particle debris) ← next
- M4 — audio pass (shravan-synthesized effects, optional slot-loaded music)
- M5 — high-score file (sankoch-compressed, sigil-integrity-hashed)
- v1.0 — end-to-end playable, polish complete

## Source

M2 complete — single level is now a five-level game. Per [ADR 0003](../adr/0003-self-rolled-primitives.md), cyrius-bb tools its own primitives on bare stdlib (cyrius-doom pattern) rather than waiting on kiran/impetus/mabda.

Present (M1 loop + M2 progression — 127 assertions green):
- `src/main.cyr` — real-time loop (tick → input → step → render → present), level-index tracking (advance on clear, game-complete on final clear) + headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion + `paddle_set_x`
- `src/bricks.cyr` — brick grid + destruction + scoring; per-cell *tier* (0=empty, 1..9), `bricks_from_cells`, `brick_tier`, `tier * value` scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` (re-serve after loss) + `world_set_bricks` (level swap)
- `src/level.cyr` — 5 original ASCII layouts, plain-text parser, auto-fit geometry, per-level speed curve, `level_make_bricks` + `level_load` (advance, score/lives carry)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()`: bricks/paddle/ball as flat rects
- `src/hud.cyr` — score + lives overlay (3×5 bitmap digit font)
- `src/input.cyr` — keyboard raw-tty input (a/d/arrows/space/q) + pure `bb_key_action` decoder
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit (untested in CI; on-console only)
- `programs/demo.cyr` — eyeball harness; dumps `build/frame00..02.ppm`

Planned modules:
- `src/audio.cyr` — sound-effect synthesis (M4)
- `src/save.cyr` — high-score file I/O (sankoch + sigil) (M5)

## Assets

Scaffold — no assets shipped yet.

Planned per [ADR 0002](../adr/0002-original-assets-only.md):
- Original sprite set (paddle, ball, brick variants) — new pixel art
- Era-spirit palette (not pixel-matching Atari Breakout's specific row colors)
- Original square-wave / FM sound effects via shravan
- 5 original level layouts landed in M2 (`src/level.cyr` — ASCII grids, ADR 0002); final art pass in M6

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — **127 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios, framebuf, render, hud, input decode, serve, level parser/tier-scoring/speed-curve, clear→advance progression). Deterministic + headless.
- Playtest gate — every milestone requires manual playthrough (feel concerns). The interactive loop + `/dev/fb0` present + the 5-level playthrough still need a real Linux console to actually playtest (build/lint + headless-smoke-verified only so far); headless logic is gated by the 127 assertions + the `<frames>` smoke.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input — on bare stdlib
- **Save / scoring**: sankoch (compression), sigil (hash) — commented out in `cyrius.cyml`, re-wired at M5
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing) — all commented out in `cyrius.cyml`. The build is now bare-stdlib, warning-free.

## Next

Start here next session (0.3.0 is the current cut):

1. **M3 — 2.5D depth pass** (v0.4.0). Parallax background (2–3 depth-offset layers), brick-destruction perspective (Z-depth tilt, not rotation), particle debris on destroy, optional impact camera shake. Per-tier brick colour now has a home (`brick_tier`) — wire it into `render.cyr`. Acceptance: reads as "2.5D arcade" vs flat 2D from a screenshot. See [`roadmap.md`](roadmap.md) M3.
2. **Console playtest** (carries from M1+M2) — verify the interactive loop + `/dev/fb0` present + the full 5-level playthrough (paddle-steer to clear each level, lose all lives, game-complete) on a real Linux console. Tune the speed-curve *feel* and add a game-over / game-complete screen (the screen is M6 polish; the loop currently just ends). Fix any blit format / resolution issues.
3. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).
4. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone — captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.4.0** — M3 (2.5D depth pass) complete. The game now reads as 2.5D arcade (orthogonal 2D + depth cues, no 3D engine): parallax background, per-tier brick colour with raised-block extrusion, brick-destruction debris + collapse particles. Built on M1 primitives + M2 levels, bare stdlib ([ADR 0003](../adr/0003-self-rolled-primitives.md)). (Prior: 0.3.0 — M2 levels; 0.2.0 — M1 loop; 0.1.0 — scaffold.)

- **DCE binary**: 111,688 B (x86_64, static, stripped) — up from 103,400 B at 0.3.0 (+8,288 for the render + fx layers).
- **Tests**: 147 assertions, 0 failed (was 127). Lint + fmt clean. Frame dumps eyeballed (parallax + tier colour + debris all present).
- **Deps**: bare stdlib — zero external deps (sankoch + sigil re-wire at M5).
- **Caveat**: interactive loop + `/dev/fb0` present + the 5-level playthrough + the *feel* of the depth effects are build/lint + headless-smoke + frame-dump-verified only (no console/framebuffer in dev/CI); logic proven via the `<frames>` smoke + 147 unit tests.

## Toolchain

- **Cyrius pin**: `6.0.1` (in `cyrius.cyml [package].cyrius`) — bumped from `5.7.11` 2026-05-25. All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5 levels, increasing speed / brick count) (✅ 0.3.0, 2026-05-26)
- M3 — 2.5D depth pass (parallax background, brick extrusion, debris + collapse) (✅ 0.4.0, 2026-05-26)
- M4 — audio pass (shravan-synthesized effects, optional slot-loaded music) ← next
- M5 — high-score file (sankoch-compressed, sigil-integrity-hashed)
- v1.0 — end-to-end playable, polish complete

## Source

M3 complete — the game reads as 2.5D. Per [ADR 0003](../adr/0003-self-rolled-primitives.md), cyrius-bb tools its own primitives on bare stdlib (cyrius-doom pattern) rather than waiting on kiran/impetus/mabda.

Present (M1 loop + M2 progression + M3 depth — 147 assertions green):
- `src/main.cyr` — real-time loop (tick → input → step → spawn-fx → render → fx → present), level-index tracking (advance on clear, game-complete on final clear) + headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion + `paddle_set_x`
- `src/bricks.cyr` — brick grid + destruction + scoring; per-cell *tier* (0=empty, 1..9), `bricks_from_cells`, `brick_tier`, `tier * value` scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` + `world_set_bricks` (level swap) + `world_last_hit`/`world_last_tier` (debris-spawn hook)
- `src/level.cyr` — 5 original ASCII layouts, plain-text parser, auto-fit geometry, per-level speed curve, `level_make_bricks` + `level_load` (advance, score/lives carry)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()` over a parallax `render_bg()`; tier-coloured bricks (`tier_rgb`) with 2px extrusion shadow
- `src/fx.cyr` — particle pool: debris shards + brick-collapse on destroy (gravity, life-shrink/dim); `fx_step`/`fx_render`
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
- Era-spirit palette landed in M3 (`tier_rgb` — original 9-tier cool→warm band, not Atari's row colours)
- Original square-wave / FM sound effects via shravan (M4)
- 5 original level layouts landed in M2 (`src/level.cyr` — ASCII grids, ADR 0002); final art pass in M6

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — **147 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios, framebuf, render + tier colour/extrusion, parallax bg, fx particle pool, hud, input decode, serve, level parser/tier-scoring/speed-curve, clear→advance progression). Deterministic + headless.
- Playtest gate — every milestone requires manual playthrough (feel concerns). The interactive loop + `/dev/fb0` present + the 5-level playthrough + the *feel* of the depth effects (parallax rate, palette, debris spread, deferred camera shake) still need a real Linux console (build/lint + headless-smoke + frame-dump-verified only so far); headless logic is gated by the 147 assertions + the `<frames>` smoke.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input — on bare stdlib
- **Save / scoring**: sankoch (compression), sigil (hash) — commented out in `cyrius.cyml`, re-wired at M5
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing) — all commented out in `cyrius.cyml`. The build is now bare-stdlib, warning-free.

## Next

Start here next session (0.4.0 is the current cut):

1. **M4 — audio pass** (v0.5.0). `src/audio.cyr` — shravan-synthesized SFX (paddle-ball blip, brick-destroy chirp, wall thud, game-over sting, level-complete fanfare), era-spirit square/FM synthesis (no samples), optional slot-loaded music at `~/.cyrius-bb/music/<level>.ogg` (silent if absent), mute toggle. NB: shravan is pinned v4.10.3 and lacks a `dist/shravan.cyr` bundle (see `cyrius.cyml` pending-deps) — it needs a toolchain bump + bundle before it can be wired; until then audio may have to self-roll a PCM `/dev/dsp`-style path or stay deferred. Acceptance: SFX rhythm matches gameplay rhythm. See [`roadmap.md`](roadmap.md) M4.
2. **Console playtest** (carries from M1–M3) — verify the interactive loop + `/dev/fb0` present + the full 5-level playthrough on a real Linux console; tune the *feel* of the M3 depth effects (parallax rate, palette, debris spread) and decide on camera shake (deferred from M3); add a game-over / game-complete screen (the screen is M6 polish; the loop currently just ends). Fix any blit format / resolution issues.
3. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).
4. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone — captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

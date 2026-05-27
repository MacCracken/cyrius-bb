# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.3.0] — 2026-05-26

**M2 — level progression.** The single level becomes a five-level game. A
plain-text layout parser turns ASCII grids into brick layouts; levels
advance on clear, each faster than the last; score and lives carry across
levels; all lives lost ends the game. 5 original layouts (ADR 0002),
increasing brick count (18 → 25 → 34 → 40 → 49). 127 headless assertions
(was 93). DCE release binary: **103,400 bytes** (was 98,648 at 0.2.0).

### Added
- `src/level.cyr` — level system: 5 embedded original ASCII layouts, a
  plain-text layout parser (`layout_cols` / `layout_rows` / `layout_count`
  / `layout_fill`, `cell_tier` char→tier map), geometry that auto-fits the
  play width, a per-level speed curve (`level_serve_vx` / `level_serve_vy`,
  vy −4 → −6 px/tick, capped below brick height so the ball can't tunnel),
  `level_make_bricks` (build a grid from a layout), and `level_load`
  (advance: swap bricks + re-serve at level speed, score/lives untouched).
- `bricks.cyr` — `bricks_from_cells` (build a grid from a parsed tier
  mask) + `brick_tier` accessor. The per-cell byte is now a *tier*
  (0 = empty, 1..9 = brick of that tier); `brick_destroy` scores
  `tier * value`, so layouts carry varied per-brick worth (and, M3, colour).
- `world.cyr` — `world_set_bricks` (swap the grid on level advance; score /
  lives / paddle carry). `paddle.cyr` — `paddle_set_x` (recentre on advance).
- `tests/` — `test_level` + `test_level_progress` (parser, tier scoring,
  grid build with gaps, speed-curve climb, and a clear→advance→play
  transition that proves score + lives carry). 34 new assertions.

### Changed
- `main.cyr` — both loops (interactive + headless `<frames>`) now track a
  level index: re-serve on loss at the current level's speed, advance to
  the next level on clear, and end on the final clear (game complete) or
  game over. `build_world` builds level 0 via the level loader instead of
  the old hardcoded 7×4 grid.
- `brick_alive` normalises to 0/1 (tiers run 1..9; `world_step` / `render`
  gate on `== 1`). M1's `bricks_new` still fills tier 1, so all 93 prior
  assertions hold unchanged.

### Feel
- Speed ramp is deliberately gentle (vy 4,4,5,5,6) to stay collision-safe;
  the curve's *feel* across the five levels is a playtest item — see the
  carried-forward console-playtest gate below.

**Carried forward** (not blocking; same gate as M1's interactive loop):
- Interactive 5-level playthrough on a real Linux console (steering the
  paddle to clear each level, losing all lives, game-complete) is
  build/lint-verified + headless-smoke-verified only — no console in
  dev/CI. The level logic is gated by the 127 assertions + the `<frames>`
  smoke; per-level *feel* and a game-over / game-complete presentation
  screen (the screen itself is M6 polish) await a console pass.

## [0.2.0] — 2026-05-25

**M1 — the foundational loop.** A playable brick-breaker built entirely on self-rolled primitives + bare Cyrius stdlib ([ADR 0003](docs/adr/0003-self-rolled-primitives.md)): deterministic fixed-point physics, an offscreen renderer, a HUD, raw-tty input, and a real-time loop. The whole simulation is unit-tested headless (93 assertions); interactive play + `/dev/fb0` present run on a Linux console. DCE release binary: **98,648 bytes** (was 748,032 at 0.1.0).

### Added
- `src/ball.cyr` — ball entity (16.16 fixed-point): position/velocity/radius + `ball_integrate`.
- `src/paddle.cyr` — paddle entity: bounded horizontal `paddle_move` + `paddle_cx` centre reference for english.
- `src/bricks.cyr` — flat cols×rows brick grid: per-brick alive flags, index→rect derivation, `brick_destroy` (scores + decrements alive count).
- `src/world.cyr` — `world_step()`: one deterministic fixed-timestep tick — integrate → wall/paddle/brick collision via `geom.cyr` → score/lives/state (playing / ball-lost / level-clear / game-over) — plus `world_serve` / `world_set_state` (re-serve after a loss).
- `src/framebuf.cyr` — offscreen RGB surface: `fb_fill_rect` (clipped), `fb_set`/`fb_get`, `fb_clear`, `fb_write_ppm` (binary P6). Pixel-assertable and CI-friendly.
- `src/render.cyr` — `render_world()`: bricks / paddle / ball as flat rects (fixed-point → pixels).
- `src/hud.cyr` — score + lives overlay via a 3×5 bitmap digit font (`hud_draw_number`).
- `src/input.cyr` — keyboard via Linux terminal raw mode (termios ioctl, non-blocking): `bb_key_action` pure decoder (unit-tested) + `input_poll` (a/d/arrows → move, space → launch, q/ESC → quit).
- `src/tick.cyr` — ~60 fps frame pacing. `src/present.cyr` — best-effort `/dev/fb0` blit (graceful no-op without a console; untested in CI by design).
- `src/main.cyr` — real-time loop (tick → input → `world_step` → `render_world` → `hud_render` → present), ball re-serve on loss, quit/clear/over handling; plus a headless `cyrius-bb <frames>` smoke mode (step N, dump a PPM, print score/bricks).
- `programs/demo.cyr` — harness that dumps `build/frame00..02.ppm` to eyeball the sim + renderer.
- `tests/cyrius-bb.tcyr` — **93 assertions** (fixed-point, geometry, ball, paddle, bricks, five `world_step` scenarios, framebuf, render, hud, input decode, serve). All green; lint + fmt clean.

### Changed
- Dropped the unused engine/asset deps from `cyrius.cyml` per [ADR 0003](docs/adr/0003-self-rolled-primitives.md): `mabda` removed (renderer is self-rolled); `sankoch` + `sigil` commented out until M5 (save file). cyrius-bb builds against **bare stdlib** — zero external deps, zero link warnings.
- DCE release binary **748,032 → 98,648 bytes** (~7.6× smaller) from removing the dormant deps.

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

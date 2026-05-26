# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

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

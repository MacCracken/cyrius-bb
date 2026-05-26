# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added
- `src/ball.cyr` — ball entity (16.16 fixed-point): position/velocity/radius struct + `ball_integrate` (position += velocity per tick).
- `src/paddle.cyr` — paddle entity: bounded horizontal `paddle_move` (clamped to the play area) + `paddle_cx` centre reference for english.
- `src/bricks.cyr` — flat cols×rows brick grid: per-brick alive flags (byte array), index→rect derivation, `brick_destroy` (scores + decrements alive count).
- `src/world.cyr` — `world_step()`: one deterministic fixed-timestep tick wiring it all together — integrate → wall/paddle/brick collision via `geom.cyr` → score/lives/state (playing / ball-lost / level-clear / game-over).
- `src/framebuf.cyr` — offscreen RGB surface (self-rolled, per ADR 0003): `fb_fill_rect` (clipped), `fb_set`/`fb_get`, `fb_clear`, and `fb_write_ppm` (binary P6 dump). No window or `/dev/fb0` needed — fully pixel-assertable and CI-friendly.
- `src/render.cyr` — `render_world()`: draws bricks / paddle / ball as flat rects (fixed-point → pixels), bricks red, paddle green, ball white.
- `programs/demo.cyr` — throwaway harness: steps a world and dumps `build/frame00..02.ppm` to eyeball the sim + renderer before the real loop lands.
- `src/input.cyr` — keyboard via Linux terminal raw mode (termios ioctl, non-blocking): `bb_key_action` pure decoder (unit-tested) + `input_poll` (a/d/arrows → move, space → launch, q/ESC → quit).
- `src/tick.cyr` — ~60 fps frame pacing (`frame_sleep`).
- `src/present.cyr` — best-effort `/dev/fb0` blit (24-bpp RGB → 32-bpp BGRX); graceful no-op when no framebuffer. Untested in CI (no console) by design.
- `src/main.cyr` — rewritten into the real-time loop (tick → input → `world_step` → `render_world` → present), with ball re-serve on loss and quit/clear/over handling. Adds a headless `cyrius-bb <frames>` smoke mode (step N, dump a PPM, print score/bricks) — CI-friendly, no I/O.
- `world_serve` / `world_set_state` in `src/world.cyr` — re-serve the ball after a life is lost.
- `tests/cyrius-bb.tcyr` — **84 assertions** (added input-decode + world-serve groups). All green; lint + fmt clean.

### Changed
- Dropped the unused engine/asset deps from `cyrius.cyml` per [ADR 0003](docs/adr/0003-self-rolled-primitives.md): `mabda` removed (renderer is self-rolled); `sankoch` + `sigil` commented out until M5 (save file). cyrius-bb now builds against **bare stdlib** — zero external deps, **zero link warnings**.
- DCE release binary: **748,032 → 114,408 bytes** (6.5× smaller) from removing the dormant deps.

M1 is now playable end-to-end on a Linux console (interactive loop) and verifiable headless (the `<frames>` smoke). Remaining M1 polish: a HUD (score/lives) and `/dev/fb0` verification on a real console.

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

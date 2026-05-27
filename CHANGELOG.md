# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

## [0.7.0] — 2026-05-26

**M6 — polish.** A title menu, pause, a beveled art pass, and a
framebuffer that adapts to real console resolutions — the last milestone
before v1.0; all core systems already exist, so this is presentation.
199 headless assertions (was 196). DCE release binary: **460,384 bytes**
(was 454,352 at 0.6.0; +6 KB for menu/art/present).

### Added
- `main.cyr` — a title-screen state machine: **menu** (BREAK BREAKER /
  PLAY / HIGH SCORES / QUIT, w-s + Enter nav via `input_nav`, `menu_move`
  wraparound) that returns to itself after each game; **pause** ('p'
  freezes the sim + shows PAUSED); the game loop is now `play_game` (fresh
  world + fx per game). High-score view reuses `render_scores`.
- `render.cyr` — final art pass (procedural, no asset files, per ADR
  0002): `draw_brick` beveled (lit top/left edge + the existing extrusion
  shadow → raised block), `draw_ball` rounded with a highlight (reads as a
  lit sphere), `draw_paddle` with a lit top edge. `lighten()` helper.
- `present.cyr` — **probes the real framebuffer geometry**
  (FBIOGET_VSCREENINFO / FSCREENINFO) and integer-scales + centres the
  surface, so it renders correctly on a normal console (e.g. 1920x1080)
  instead of filling only the top-left corner. Still best-effort / not
  CI-tested; graceful no-op without `/dev/fb0`.
- `input.cyr` — `input_nav` (menu navigation, arrow + w/s/Enter/q decode)
  and `ACT_PAUSE` ('p').
- `tests/` — `test_render` updated for the beveled/rounded art (bevel
  highlight, interior face, paddle edge + body, ball body + highlight). 3
  net new assertions.

### Accessibility
- Keyboard-only play (already true) is now also keyboard-only menus.
- Brick legibility no longer rests on hue alone: the bevel adds a
  shape/highlight cue and tier maps to row position, so colour-blind
  players can distinguish rows even where hues are close. The 9-tier
  palette keeps brightness variation; a full CVD-optimized repalette is a
  playtest-gated tweak, not done here.

**Carried forward** (the pre-v1.0 playtest gate — you're driving this):
- Console playthrough of the whole flow (menu → play → pause → game over →
  high-score entry → menu) on a real Linux console + `/dev/dsp` audio.
  Tune feel (depth/SFX/speed), verify the framebuffer scale/centre on the
  actual console resolution, and decide camera shake. `present.cyr`'s
  ioctl path is the highest-risk untested code — most likely to need a
  morning fix.
- Binary size (~460 KB) unchanged in character from M5 ([note 002](docs/architecture/002-save-deps-binary-size.md)).

## [0.6.0] — 2026-05-26

**M5 — high-score persistence.** A top-10 table with 3-letter initials,
saved to `~/.cyrius-bb/scores.cyb`: sankoch-compressed + sigil-HMAC-SHA256
integrity-hashed, tamper-rejecting. This is the first milestone to consume
the shared crates rather than self-roll (ADR 0003 — "use them, don't
self-roll zip/crypto"). 196 headless assertions (was 190). DCE release
binary: **454,352 bytes** — up from 117,184 at 0.5.0; the ~4x jump is the
cost of compiling in sankoch + sigil, which DCE can't trim (see
[architecture note 002](docs/architecture/002-save-deps-binary-size.md)).

### Added
- `src/save.cyr` — the persistence engine (pure + buffer-based, so it is
  headless-tested): top-10 table (`hs_insert` keeps it sorted descending,
  drops the overflow entry; `hs_qualifies`), a versioned serializer, and
  `hs_encode`/`hs_decode` — payload → sankoch zlib + a sigil HMAC-SHA256
  over the plaintext, in a `CYBB`-magic container. `hs_decode` rejects a
  bad magic / size (-1) or HMAC mismatch (-2) and leaves the table
  untouched, so a hand-edited file is refused, not trusted. `hs_save` /
  `hs_load` are the thin `~/.cyrius-bb/scores.cyb` file wrappers (best-
  effort `mkdir`, `$HOME` via `getenv`; first run with no file is rc=1,
  not an error).
- `hud.cyr` — an A-Z 3x5 font (`hud_draw_letter` / `hud_draw_char`) + a
  `hud_draw_text` string drawer, so initials + names render. `glyph()`
  helper factors the bit-packing.
- `input.cyr` — `input_read_byte` (raw non-blocking byte) for initials
  entry.
- `main.cyr` — high-score table loaded at start-up; on a natural finish
  (game over / all levels cleared, not a quit) a qualifying score opens an
  initials-entry screen, inserts + saves, then shows the table until a
  keypress. Console-only (not CI-tested), like the rest of the loop.
- `programs/scores_demo.cyr` — dumps the font + a sample table to PPM (the
  eyeball harness, like `demo`/`audio_demo`); used to verify the glyphs.
- `docs/architecture/002-save-deps-binary-size.md`.
- `tests/` — `test_scores` (qualify/insert ordering + overflow cap,
  serialize round-trip, zlib+HMAC encode/decode round-trip, tamper → -2,
  bad-magic → -1) + letter-font assertions. 23 new assertions.

### Changed
- `cyrius.cyml` — wired the M5 deps: sankoch + sigil (+ their transitive
  freelist / bigint / ct / keccak) added to `[deps].stdlib`. They ship in
  the 6.0.1 toolchain snapshot, so they resolve from the pinned lib with
  no git fetch; the commented git-source blocks are kept for reference.
  `main()` / tests call `fl_init()` (sigil's allocator) at start-up.

### Feel
- The score-entry + high-score screens are first-pass console UI; their
  layout/feel and a proper game-over/menu flow are playtest + M6 territory.

**Carried forward** (not blocking):
- Binary size: the 4x jump is accepted as the honest cost of real
  compression + crypto; leaner consumption of the bundles is an upstream
  DCE-friendliness concern, revisited only if size becomes a blocker.
- Carries from M1–M4: full console playthrough + the entry/high-score UI
  on a real console, depth/audio feel, audible `/dev/dsp`, camera shake,
  and a polished game-over/menu (M6).

## [0.5.0] — 2026-05-26

**M4 — the audio pass.** Era-spirit square-wave SFX, self-rolled on bare
stdlib — shravan has no consumable bundle and ALSA/Pulse/PipeWire need FFI
(banned), so cyrius-bb synthesises its own sound the way it self-rolled the
renderer ([ADR 0003](docs/adr/0003-self-rolled-primitives.md); rationale in
[architecture note 001](docs/architecture/001-no-ffi-audio.md)). Square
beeps are faithful to Breakout's 1976 sound anyway. 166 headless assertions
(was 147). DCE release binary: **117,184 bytes** (was 111,688 at 0.4.0).

### Added
- `src/audio.cyr` — SFX synthesis: `synth_tone` (square wave, linear
  frequency + amplitude sweep, unsigned 8-bit mono @ 11025 Hz) and six
  prebuilt SFX (paddle blip, brick chirp, wall thud, ball-lost blip,
  game-over sting, level-complete arpeggio). The *tested* core — no device
  I/O, sample-assertable (mirrors `framebuf.cyr`). Plus `audio_write_wav`
  (dump a buffer to a playable WAV) and a mute flag.
- `src/sound.cyr` — best-effort OSS `/dev/dsp` sink (write-only,
  non-blocking, requests the synth rate via `SNDCTL_DSP_SPEED`). The
  `present.cyr` analogue: environment-specific, **not unit-tested**, a
  silent no-op if the device is unavailable.
- `programs/audio_demo.cyr` — dumps all six SFX to `build/sfx_*.wav` so the
  sounds can be heard / verified without a device (the ear-equivalent of
  `demo.cyr`'s eyeball frames).
- `world.cyr` — `world_events` collision-event bitmask (`EV_WALL` /
  `EV_PADDLE` / `EV_BRICK`), set per tick and cleared each step — the audio
  trigger hook. `input.cyr` — `ACT_MUTE` ('m').
- `docs/architecture/001-no-ffi-audio.md` — the no-FFI / no-OGG-decoder
  constraint and its consequences.
- `tests/` — `test_audio` (square-wave sample pattern, amp-0 silence, amp
  envelope, SFX registry, mute toggle), `ACT_MUTE` decode, and `EV_*`
  assertions on the wall/paddle/brick world-step tests. 19 new assertions.

### Changed
- `main.cyr` — `audio_init` at start-up; the interactive loop opens the
  sink, fires SFX off the per-tick event bitmask + state transitions
  (ball-lost, game-over, level-complete), toggles mute on 'm', and closes
  the sink on exit. Headless mode stays silent (no device).

### Feel
- SFX params (frequencies, durations, envelopes) and the lack of a voice
  mixer are playtest territory — the synthesis is verified (WAV dumps +
  166 assertions), but whether the SFX *rhythm matches the gameplay rhythm*
  wants a console ear. Tune in the playtest pass.

**Carried forward** (not blocking):
- **Music** (slot-loaded `.ogg`, roadmap M4) is **deferred** — OGG decode
  is infeasible without FFI / a decoder; if revisited it will be WAV. The
  SFX-focused M4 acceptance does not depend on it.
- **Audible playback on a real device.** Modern desktops lack `/dev/dsp`;
  needs an OSS shim (`padsp ./cyrius-bb`) or an OSS console — same
  build-verified-only status as `/dev/fb0` present. WAV dumps are the
  always-works verification path.
- Carries from M1–M3: full console playthrough, depth-effect feel, camera
  shake, game-over/complete screen (M6).

## [0.4.0] — 2026-05-26

**M3 — the 2.5D depth pass.** The game now *reads* as 2.5D, not flat 2D
(orthogonal 2D + depth cues — still no 3D engine, per the scope rule): a
parallax background, per-tier brick colour with a raised-block extrusion,
and brick-destruction debris + collapse particles. 147 headless assertions
(was 127). DCE release binary: **111,688 bytes** (was 103,400 at 0.3.0).

### Added
- `src/fx.cyr` — transient visual-effects layer: a deterministic
  fixed-capacity particle pool, kept separate from the sim `world` so
  physics stays clean. `fx_debris` throws 5 colour-matched shards on a
  brick hit; `fx_collapse` emits one brick-sized shard that shrinks to a
  point (the destroyed brick receding in Z). Particles drift under gravity,
  shrink + dim with remaining life, and free their slot on expiry.
  `fx_step` / `fx_render` drive + draw the pool.
- `render.cyr` — `render_bg`: parallax background (three vertical depth
  zones + a far/slow and near/fast vertical-bar layer that shift with the
  ball's x at different rates — the depth cue). `tier_rgb` (original 9-tier
  cool→warm palette, ADR 0002 — *not* Atari's row colours) + `rgb_r/g/b`
  extractors. Bricks now draw a 2px bottom-right extrusion shadow under a
  tier-coloured face, so they read as raised blocks.
- `world.cyr` — `world_last_hit` / `world_last_tier`: a render-loop hook
  exposing the brick destroyed this tick (idx + tier, cleared to -1 each
  step) so the loop can spawn debris at the right spot + colour. Does not
  affect the deterministic sim.
- `tests/` — `test_colors`, `test_bg` (parallax shifts with ball x),
  `test_fx` (spawn / capacity cap / integration + gravity / expiry); 20 new
  assertions, plus `test_render` updated for tier colour + extrusion.

### Changed
- `main.cyr` — both loops create an `fx` pool, spawn destruction effects
  from `world_last_hit` (before any level swap), and step + draw the fx
  layer over the world frame each tick.
- `render_world` now paints `render_bg` instead of a flat clear, and
  colours bricks by tier.

### Feel
- Camera shake on impact (the one optional M3 bullet) is **deferred to the
  console-playtest pass** — it is not screenshot-visible (so it doesn't
  move the M3 "tell it's 2.5D from a screenshot" bar) and is best tuned
  live. The parallax rates, palette, and debris spread likewise want a
  playtest eye; the effects are headless-verified (frame dumps + 147
  assertions) but their *feel* is a console concern.

**Carried forward** (not blocking; same gate as M1/M2):
- Interactive console playthrough (loop + `/dev/fb0` present, 5-level run,
  feel of the depth effects + speed curve, camera shake) — build/lint +
  headless-smoke + frame-dump-verified only; no console in dev/CI.

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

# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.7.2** — toolchain bump + game-loop review pass. Pins Cyrius `6.0.1 → 6.2.2` and acts on the two game-loop issues from the first console playtest (after 0.7.1's input fix): **(1)** paddle hold-to-move lurched (raw-tty key-repeat read as held state) — fixed with a decaying key-held latch (`input_hold_step`) so it glides every frame; **(2)** ball speed wasn't conserved on paddle bounce (edge hits sped it up → level 1 too fast) — fixed with a speed-conserving angle-zone rebound (`paddle_rebound_vx/vy`) plus a per-level `level_speed` ramp (5.0 → 7.0) as the difficulty knob. Mechanism unit-tested (222 green, was 199); *feel* numbers (latch length, zone angles, speed ramp) are console-playtest knobs. Brick side-hit axis + camera shake stay deferred ([`roadmap.md`](roadmap.md)). Binary grew with the toolchain (see below). (Prior: 0.7.1 — console-input fix; 0.7.0 — M6 polish; 0.6.0 — M5 scores; 0.5.0 — M4 audio; 0.4.0 — M3 depth; 0.3.0 — M2 levels; 0.2.0 — M1 loop; 0.1.0 — scaffold.)

- **DCE binary**: **1,114,888 B** (x86_64, static, stripped) — up 2.4× from 460,384 B at 0.7.0/0.7.1 with the `6.0.1 → 6.2.2` toolchain bump. ~+533 KB is 6.2.2's larger folded stdlib bundles (sankoch/sigil/keccak/ct); ~+120 KB is the `bayan` bundle (carved-out `bigint`/`u128`), kept for a warning-free build — its `u256_*` refs back sigil's otherwise-unreachable signature path. One residual advisory: `large static data (335 KB)` from the stdlib bundles' lookup tables (not game code — the framebuffer is heap-allocated). The save-deps cost remains the bulk ([note 002](../architecture/002-save-deps-binary-size.md)); M1–M4 were ~100 KB bare-stdlib.
- **Tests**: 222 assertions, 0 failed (was 199 — +23 for the 0.7.2 game-loop pass: rebound speed-conservation across the english range, the per-level speed curve, and the `input_hold_step` latch). Lint + fmt clean against 6.2.2. (The raw-tty *device* path + the *feel* of the latch/rebound are still device/playtest-only — the suite covers the pure cores.)
- **Deps**: stdlib + sankoch + sigil (+ transitive freelist/**bayan**/ct/keccak/thread_local), all from the 6.2.2 toolchain snapshot — no git fetch. `bayan` replaces the 6.0.1 `bigint` (carved into the bayan bundle); `thread_local` now listed explicitly (no longer pulled transitively with `thread`). shravan stays deferred (no consumable bundle; audio self-rolled).
- **Caveat**: the whole interactive flow (menu → play → pause → game-over → high-score entry) + `/dev/fb0` present (now resolution-probing) + audible `/dev/dsp` + the *feel* of depth/audio/speed are build/lint + headless-smoke + frame/WAV-dump-verified only (no console/framebuffer/audio device in dev/CI). Headless logic is gated by 222 unit tests + the `<frames>` smoke; the save engine by an out-of-band disk round-trip. **First real-VT run (0.7.1) confirmed the framebuffer present works and fixed the blocking-input bug; a full end-to-end console playthrough remains the open pre-v1.0 gate.** Known remaining console gap: no `KDSETMODE`/`KD_GRAPHICS` VT switch, so `fbcon` text + cursor can fight the blit.

## Toolchain

- **Cyrius pin**: `6.2.2` (in `cyrius.cyml [package].cyrius`) — bumped from `6.0.1` 2026-06-13 (was `5.7.11` → `6.0.1` 2026-05-25). The bump needed two manifest edits beyond the pin number: 6.1.x's "bayan distfile carve" moved `bigint`/`u128` (+ json/toml/csv/base64) out of stdlib into the **bayan** bundle, so `[deps].stdlib` now lists `bayan` where it listed `bigint`; and 6.2.2's resolver no longer transitively pulls `thread_local` with `thread`, so it's listed explicitly (sigil's HMAC path calls `thread_local_get/set`). `lib/` regenerated against 6.2.2 (72 → 42 vendored modules). 222 assertions green, lint clean.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5 levels, increasing speed / brick count) (✅ 0.3.0, 2026-05-26)
- M3 — 2.5D depth pass (parallax background, brick extrusion, debris + collapse) (✅ 0.4.0, 2026-05-26)
- M4 — audio pass (self-rolled square-wave SFX + OSS sink + mute; music deferred) (✅ 0.5.0, 2026-05-26)
- M5 — high-score file (sankoch-compressed, sigil-HMAC-hashed, tamper-rejecting) (✅ 0.6.0, 2026-05-26)
- M6 — polish (menu, pause, framebuffer scaling, beveled art, accessibility) (✅ 0.7.0, 2026-05-26)
- v1.0 — end-to-end playable, polish complete ← next (after the console playtest gate; target 2026-06-13)

## Source

M6 complete — full game shell. Self-rolled on bare stdlib through M4 ([ADR 0003](../adr/0003-self-rolled-primitives.md)); M5 added the shared crates (sankoch + sigil); M6 is presentation polish over the existing systems.

Present (M1 loop + M2 levels + M3 depth + M4 audio + M5 scores + M6 polish — 222 assertions green):
- `src/main.cyr` — title menu (play / high scores / quit, `menu_loop`/`menu_move`) → `play_game` (real-time loop: tick → input → step → spawn-fx → SFX → render → fx → present, level advance, pause, mute) → high-score entry/table; headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english + speed-conserving `paddle_rebound_vx/vy` (discrete angle zones, ADR 0001)
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion + `paddle_set_x`
- `src/bricks.cyr` — brick grid + destruction + scoring; per-cell *tier* (0=empty, 1..9), `bricks_from_cells`, `brick_tier`, `tier * value` scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` + `world_set_bricks` (level swap) + `world_speed`/`world_set_speed` (per-level target, paddle bounce re-asserts |v|) + `world_last_hit`/`world_last_tier` (debris hook) + `world_events` (SFX hook)
- `src/level.cyr` — 5 original ASCII layouts, plain-text parser, auto-fit geometry, `level_speed` per-level target (5.0 → 7.0) + serve curve, `level_make_bricks` + `level_load` (advance, score/lives carry)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()` over a parallax `render_bg()`; tier-coloured (`tier_rgb`) beveled bricks (`draw_brick`), rounded highlighted ball (`draw_ball`), lit-edge paddle (`draw_paddle`)
- `src/fx.cyr` — particle pool: debris shards + brick-collapse on destroy (gravity, life-shrink/dim); `fx_step`/`fx_render`
- `src/audio.cyr` — square-wave SFX synthesis (`synth_tone`, 6 SFX) + WAV dump + mute; the tested, device-free core
- `src/sound.cyr` — best-effort OSS `/dev/dsp` SFX sink (untested in CI; device/console only)
- `src/save.cyr` — high-score table: insert/qualify/serialize + `hs_encode`/`hs_decode` (sankoch zlib + sigil HMAC-SHA256, tamper-rejecting) + `hs_save`/`hs_load` (`~/.cyrius-bb/scores.cyb`). Engine is buffer-pure + tested
- `src/hud.cyr` — score + lives overlay; 3×5 digit + A-Z font (`hud_draw_letter`/`hud_draw_char`/`hud_draw_text`)
- `src/input.cyr` — raw-tty input: `bb_key_action` (a/d/arrows/space/q/m/p) + `input_poll`, `input_read_byte` (initials), `input_nav` (menu) + `input_hold_step` (decaying key-held latch so hold-to-move glides across key-repeat gaps)
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit, probes geometry + integer-scales + centres (untested in CI; on-console only)
- `programs/demo.cyr` — frame eyeball harness (`build/frame00..02.ppm`); `programs/audio_demo.cyr` — SFX ear harness (`build/sfx_*.wav`); `programs/scores_demo.cyr` — font + table eyeball harness

All milestones M0–M6 are in; v1.0 is the polish-complete + playtested release, not new systems.

## Assets

Scaffold — no assets shipped yet.

Planned per [ADR 0002](../adr/0002-original-assets-only.md):
- Original sprite set (paddle, ball, brick variants) — new pixel art
- Era-spirit palette landed in M3 (`tier_rgb` — original 9-tier cool→warm band, not Atari's row colours)
- Original square-wave SFX landed in M4 (`src/audio.cyr` — self-rolled, no shravan; see [note 001](../architecture/001-no-ffi-audio.md))
- 5 original level layouts landed in M2 (`src/level.cyr` — ASCII grids, ADR 0002); final art pass in M6

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — **222 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios + collision events, framebuf, render + beveled-art/parallax, fx particle pool, audio synth/mute, hud digits + letters, input decode, serve, level parser/tier-scoring/speed-curve, clear→advance progression, high-score qualify/insert/serialize/encode-decode/tamper). Deterministic + headless.
- Playtest gate — the full interactive flow (menu → play → pause → game-over → high-score entry) + `/dev/fb0` present (now resolution-probing — verify scale/centre on the real console) + audible `/dev/dsp` + the *feel* of depth/audio/speed (parallax rate, palette, debris spread, SFX rhythm, deferred camera shake) need a real Linux console + audio device. Build/lint + headless-smoke + frame/WAV-dump-verified only so far; logic gated by 222 assertions + the `<frames>` smoke, save engine by an out-of-band disk round-trip. `present.cyr`'s ioctl probe is the highest-risk untested path.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input, audio synth + OSS sink — on bare stdlib
- **Save / scoring** (WIRED at M5): sankoch (compression) + sigil (HMAC hash), + transitive freelist/bayan/ct/keccak/thread_local — all in `[deps].stdlib`, resolved from the 6.2.2 toolchain snapshot (no git fetch; `bayan` is the carved-out former `bigint`). Cost: ~2.4x binary at 6.2.2 ([note 002](../architecture/002-save-deps-binary-size.md)).
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing), shravan (audio — no consumable bundle; self-rolled instead, see [note 001](../architecture/001-no-ffi-audio.md)) — all commented out in `cyrius.cyml`. The build is warning-free.

## Next

Start here next session (0.7.0 is the current cut — all milestones M0–M6 done):

1. **Console playtest** (THE pre-v1.0 gate — user-driven, planned for the morning after 0.7.0). On a real Linux console, run the whole flow: menu → play 5 levels → pause → lose all lives / clear all → high-score entry → table → back to menu. Verify `/dev/fb0` present scales + centres correctly at the console's resolution (the highest-risk untested code — `present.cyr` ioctl probe); audible `/dev/dsp` (likely `padsp ./cyrius-bb`; [note 001](../architecture/001-no-ffi-audio.md)). Tune *feel*: ball speed curve, paddle english, parallax rate, palette, debris spread, SFX rhythm; decide camera shake (deferred from M3) + a voice mixer if SFX overlap badly. Capture issues → fix.
2. **v1.0 close** — once the playtest is clean: closeout pass (CLAUDE.md), final size note, CI matrix green, then cut v1.0 (target **2026-06-13**, comfortably ahead). Knife article outline at [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md).
3. **Binary size** (optional) — ~460 KB since M5 ([note 002](../architecture/002-save-deps-binary-size.md)); revisit only if it becomes a release blocker (leaner sankoch/sigil consumption is upstream work).
4. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).

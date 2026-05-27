# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.6.0** — M5 (high-score persistence) complete. Top-10 table with 3-letter initials at `~/.cyrius-bb/scores.cyb`: sankoch-compressed + sigil-HMAC-SHA256 integrity-hashed, tamper-rejecting. First milestone to USE the shared crates rather than self-roll (ADR 0003). Built on M1 loop + M2 levels + M3 depth + M4 audio. (Prior: 0.5.0 — M4 audio; 0.4.0 — M3 depth; 0.3.0 — M2 levels; 0.2.0 — M1 loop; 0.1.0 — scaffold.)

- **DCE binary**: 454,352 B (x86_64, static, stripped) — up from 117,184 B at 0.5.0. The ~4x jump is the cost of compiling in sankoch + sigil; DCE can't trim them (see [architecture note 002](../architecture/002-save-deps-binary-size.md)). The bare-stdlib milestones (M1–M4) stayed ~100 KB; M5 integrates the heavyweight crates by design.
- **Tests**: 196 assertions, 0 failed (was 166). Lint + fmt clean. Disk round-trip verified out-of-band (save→reload across processes; hand-edit → rejected; first-run → graceful).
- **Deps**: stdlib + sankoch + sigil (+ transitive freelist/bigint/ct/keccak), all from the 6.0.1 toolchain snapshot — no git fetch. shravan stays deferred (no consumable bundle; audio self-rolled).
- **Caveat**: interactive loop + `/dev/fb0` present + audible `/dev/dsp` + the 5-level playthrough + the score-entry/high-score UI + the *feel* of depth/audio are build/lint + headless-smoke + frame/WAV-dump-verified only (no console/framebuffer/audio device in dev/CI); the save engine + logic are proven via the `<frames>` smoke + 196 unit tests + an out-of-band disk round-trip.

## Toolchain

- **Cyrius pin**: `6.0.1` (in `cyrius.cyml [package].cyrius`) — bumped from `5.7.11` 2026-05-25. All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5 levels, increasing speed / brick count) (✅ 0.3.0, 2026-05-26)
- M3 — 2.5D depth pass (parallax background, brick extrusion, debris + collapse) (✅ 0.4.0, 2026-05-26)
- M4 — audio pass (self-rolled square-wave SFX + OSS sink + mute; music deferred) (✅ 0.5.0, 2026-05-26)
- M5 — high-score file (sankoch-compressed, sigil-HMAC-hashed, tamper-rejecting) (✅ 0.6.0, 2026-05-26)
- M6 — polish (menu, pause, screen sizing, final art, accessibility) ← next
- v1.0 — end-to-end playable, polish complete

## Source

M5 complete — scores persist. Self-rolled on bare stdlib through M4 ([ADR 0003](../adr/0003-self-rolled-primitives.md)); M5 is the first milestone to consume the shared crates (sankoch + sigil) rather than self-roll.

Present (M1 loop + M2 levels + M3 depth + M4 audio + M5 scores — 196 assertions green):
- `src/main.cyr` — real-time loop (tick → input → step → spawn-fx → SFX → render → fx → present), level-index tracking (advance on clear, game-complete on final clear), mute toggle, high-score load + score-entry/table screens, headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion + `paddle_set_x`
- `src/bricks.cyr` — brick grid + destruction + scoring; per-cell *tier* (0=empty, 1..9), `bricks_from_cells`, `brick_tier`, `tier * value` scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` + `world_set_bricks` (level swap) + `world_last_hit`/`world_last_tier` (debris hook) + `world_events` (SFX hook)
- `src/level.cyr` — 5 original ASCII layouts, plain-text parser, auto-fit geometry, per-level speed curve, `level_make_bricks` + `level_load` (advance, score/lives carry)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()` over a parallax `render_bg()`; tier-coloured bricks (`tier_rgb`) with 2px extrusion shadow
- `src/fx.cyr` — particle pool: debris shards + brick-collapse on destroy (gravity, life-shrink/dim); `fx_step`/`fx_render`
- `src/audio.cyr` — square-wave SFX synthesis (`synth_tone`, 6 SFX) + WAV dump + mute; the tested, device-free core
- `src/sound.cyr` — best-effort OSS `/dev/dsp` SFX sink (untested in CI; device/console only)
- `src/save.cyr` — high-score table: insert/qualify/serialize + `hs_encode`/`hs_decode` (sankoch zlib + sigil HMAC-SHA256, tamper-rejecting) + `hs_save`/`hs_load` (`~/.cyrius-bb/scores.cyb`). Engine is buffer-pure + tested
- `src/hud.cyr` — score + lives overlay; 3×5 digit + A-Z font (`hud_draw_letter`/`hud_draw_char`/`hud_draw_text`)
- `src/input.cyr` — keyboard raw-tty input (a/d/arrows/space/q/m) + pure `bb_key_action` decoder + `input_read_byte` (initials entry)
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit (untested in CI; on-console only)
- `programs/demo.cyr` — frame eyeball harness (`build/frame00..02.ppm`); `programs/audio_demo.cyr` — SFX ear harness (`build/sfx_*.wav`); `programs/scores_demo.cyr` — font + table eyeball harness

All planned game modules now exist; M6 is polish (menu/pause/screens/art), not new core systems.

## Assets

Scaffold — no assets shipped yet.

Planned per [ADR 0002](../adr/0002-original-assets-only.md):
- Original sprite set (paddle, ball, brick variants) — new pixel art
- Era-spirit palette landed in M3 (`tier_rgb` — original 9-tier cool→warm band, not Atari's row colours)
- Original square-wave SFX landed in M4 (`src/audio.cyr` — self-rolled, no shravan; see [note 001](../architecture/001-no-ffi-audio.md))
- 5 original level layouts landed in M2 (`src/level.cyr` — ASCII grids, ADR 0002); final art pass in M6

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — **196 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios + collision events, framebuf, render + tier colour/extrusion, parallax bg, fx particle pool, audio synth/mute, hud digits + letters, input decode, serve, level parser/tier-scoring/speed-curve, clear→advance progression, high-score qualify/insert/serialize/encode-decode/tamper). Deterministic + headless.
- Playtest gate — every milestone requires manual playthrough (feel concerns). The interactive loop + `/dev/fb0` present + audible `/dev/dsp` + the 5-level playthrough + the score-entry/high-score UI + the *feel* of depth/audio (parallax rate, palette, debris spread, SFX rhythm, deferred camera shake) still need a real Linux console + audio device (build/lint + headless-smoke + frame/WAV-dump-verified only so far); headless logic is gated by the 196 assertions + the `<frames>` smoke, and the save engine by an out-of-band disk round-trip.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input, audio synth + OSS sink — on bare stdlib
- **Save / scoring** (WIRED at M5): sankoch (compression) + sigil (HMAC hash), + transitive freelist/bigint/ct/keccak — all in `[deps].stdlib`, resolved from the 6.0.1 toolchain snapshot (no git fetch). Cost: ~4x binary ([note 002](../architecture/002-save-deps-binary-size.md)).
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing), shravan (audio — no consumable bundle; self-rolled instead, see [note 001](../architecture/001-no-ffi-audio.md)) — all commented out in `cyrius.cyml`. The build is warning-free.

## Next

Start here next session (0.6.0 is the current cut):

1. **M6 — polish** (v0.9.0). Menu screen (title / play / high scores / quit), pause, screen-size / windowed-mode handling, final art pass (replace placeholder solid-colour rects per ADR 0002), accessibility (keyboard-only — already true; colour-blind-friendly tier palette review). The score-entry + high-score screens from M5 fold into the menu's "high scores" view; the game-over flow gets a proper screen. See [`roadmap.md`](roadmap.md) M6. NB: all core systems exist — M6 is presentation/polish, the last stop before v1.0 (target 2026-06-13).
2. **Console + device playtest** (carries from M1–M5; the big pre-v1.0 gate) — on a real Linux console: the interactive loop + `/dev/fb0` present + audible `/dev/dsp` (likely `padsp ./cyrius-bb`; [note 001](../architecture/001-no-ffi-audio.md)) + full 5-level playthrough + score-entry/high-score UI. Tune feel: depth effects, SFX rhythm + possible voice mixer, camera shake (deferred from M3). Fix any blit/audio format issues.
3. **Binary size** (optional) — 454 KB since M5 ([note 002](../architecture/002-save-deps-binary-size.md)); revisit only if it becomes a release blocker (would need leaner sankoch/sigil consumption upstream).
4. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).
4. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone — captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

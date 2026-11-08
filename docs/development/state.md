# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.9.2** — code audit sweep and repair ([report](../audit/2026-11-08-audit.md)): no CRITICAL or HIGH findings, 3 MEDIUM + 12 LOW, all fixed. The input layer is one stateful decoder, post-game screens wait for held keys to stop, saves are atomic, non-terminal stdin exits cleanly, and brick side hits flip `vx` (**a gameplay change: a bot's clears ran about a third longer; playtest pending**). 382 assertions green; lint + fmt clean. (Prior: 0.9.1 — toolchain `6.7.9 → 6.7.18` (sankoch 2.8.9, sigil 3.13.15, bayan 1.11.1); 0.9.0 — toolchain `6.6.20 → 6.7.9` + vani-core `1.2.9 → 1.2.10`; 0.8.4 — the sound fix (below) + toolchain `6.6.6 → 6.6.20` + vani-core `1.2.5 → 1.2.9`; 0.8.3 — toolchain `6.6.2 → 6.6.6` + vani-core `0.9.9 → 1.2.5`; 0.8.2 — toolchain `6.3.5 → 6.6.2`; 0.8.1 — vani-core 0.9.6 → 0.9.9; 0.8.0 — audio moves to ALSA via vani, toolchain `6.2.2 → 6.3.5`; 0.7.2 — game-loop review; 0.7.1 — console-input fix; 0.7.0 — M6 polish; 0.6.0 — M5 scores; 0.5.0 — M4 audio; 0.4.0 — M3 depth; 0.3.0 — M2 levels; 0.2.0 — M1 loop; 0.1.0 — scaffold.)

**Sound actually plays** (fixed 2026-09-26, shipped in 0.8.4). A hardware probe showed the in-game audio had been silent since the 0.8.0 ALSA port: card 1's ALC897 refused the synth's 11025 Hz 8-bit mono with `-EINVAL`, and the return went unchecked. `src/audio.cyr` now renders 48 kHz S16_LE stereo. `src/sound.cyr` opens an explicit 32768-frame ring with silence-filled sw params, re-prepares on XRUN, and stays closed (silent) if the device refuses the format. The device path is verified on card 1 with zeroed PCM; **the listening test is still pending (Next item 1).** Detail in CHANGELOG `[0.8.4]`.

- **DCE binary**: **1,165,272 B** (x86_64, static, stripped), up from 1,156,752 B at 0.9.1 (+8,520 B, +0.74%, the audit's repairs). 0.9.1's own step up from 1,019,000 B (+137,752 B, +13.5%) is attributed by staged builds: the 6.7.18 compiler −12,288 B, sankoch 2.8.9 +67,040 B, sigil 3.13.15 −264,872 B (its 262,144 B static crypto bank is gone), **bayan 1.11.1 +348,296 B**, the rest of the stdlib −424 B. Only 22 of bayan's 1,118 functions survive DCE and `.rodata` grew 342,250 B, so bayan's cost is static data that function-level DCE cannot drop, most likely the Unicode tables it now requires. The earlier large step was 0.8.3's (832,152 → 999,864 B), mostly **sankoch 2.8.0's new Brotli decoder at +171,440 B**: `src/save.cyr` calls the generic `decompress()` dispatcher, so DCE has to keep the decoder and its 122,784 B static dictionary. One residual advisory remains: `large static data (153 KB)`, from the stdlib bundles' tables rather than game code (the framebuffer is heap-allocated). The save deps are still the bulk ([note 002](../architecture/002-save-deps-binary-size.md)); M1–M4 were ~100 KB bare-stdlib. (For history: 1,366,808 B at 0.8.0 on 6.3.5, and 832,152 B at 0.8.2 on 6.6.2.)
- **Tests**: **382** assertions at 0.9.2 (+129 from the audit, plus a real fuzz harness over the save and input decoders, in CI); 222 at 0.8.3, 253 from 0.8.4 (+31 for the audio fix: the 48 kHz S16 synth, plus the sound shell's retry policy and ring sizing), 0 failed. (222 was 199 — +23 for the 0.7.2 game-loop pass: rebound speed-conservation across the english range, the per-level speed curve, and the `input_hold_step` latch). Lint + fmt clean against 6.7.18. (The raw-tty *device* path + the *feel* of the latch/rebound are still device/playtest-only — the suite covers the pure cores.)
- **Deps**: stdlib + sankoch 2.8.9 + sigil 3.13.15 (+ transitive freelist/**bayan** 1.11.1/ct/keccak/thread_local/random/**sys**), all from the 6.7.18 toolchain snapshot, with no git fetch. `sys` was added at 0.8.3 because sigil calls `sys_uname`. `thread_local` and the `bench` harness have been manual-include modules since 6.3.x (included explicitly in `src/main.cyr` and the test/bench/fuzz entry points). Plus the vendored **vani-core** 1.2.10 ALSA shim (`vendor/vani-core.cyr`, a committed single-file snapshot, not a git dep; 1.2.10 is what 6.7.18 folds as `lib/vani.cyr`). The build's only warning is the static-data advisory. shravan stays deferred (ADR 0003; a `dist/` bundle exists at 2.9.3, but nothing here needs it).
- **Caveat**: the whole interactive flow (menu → play → pause → game-over → high-score entry) + `/dev/fb0` present (now resolution-probing) + audible ALSA (`/dev/snd`, via vani-core; the device path was hardware-probed on 2026-09-26, but nobody has listened yet) + the *feel* of depth/audio/speed are build/lint + headless-smoke + frame/WAV-dump-verified only (no console/framebuffer in dev/CI). Headless logic is gated by 382 unit tests + the `<frames>` smoke; the save engine by an out-of-band disk round-trip. **First real-VT run (0.7.1) confirmed the framebuffer present works and fixed the blocking-input bug; a full end-to-end console playthrough remains the open pre-v1.0 gate.** Known remaining console gap: no `KDSETMODE`/`KD_GRAPHICS` VT switch, so `fbcon` text + cursor can fight the blit.

## Toolchain

- **Cyrius pin**: `6.7.18` (in `cyrius.cyml [package].cyrius`), bumped from `6.7.9` on 2026-11-08. `lib/` was re-vendored with `cyrius deps` (62 modules, byte-identical to the snapshot; a resolve from an empty `lib/` gives the same file set and `cyrius.lock`). New at this bump: `lib/math.cyr` and `lib/unicode/`, which bayan 1.11 requires. `[deps].stdlib` is unchanged. 382 assertions green, lint + fmt clean. (History: `6.6.20 → 6.7.9` 2026-11-08; `6.6.6 → 6.6.20` 2026-11-08, when `cyrius.lock` gained real entries and CI's `deps --verify` started running; `6.6.2 → 6.6.6` 2026-09-26, a mandatory re-vendor at the 6.6.5 boundary because the aarch64 syscall peer moved `SYS_UNLINKAT`, and `sys` added to `[deps].stdlib`; `6.3.5 → 6.6.2` 2026-09-11; `6.2.2 → 6.3.5` 2026-06-29, which added `random` and made `thread_local` + `bench` explicit includes; `6.0.1 → 6.2.2` 2026-06-13; `5.7.11 → 6.0.1` 2026-05-25. The 6.1.x "bayan distfile carve" replaced `bigint` with the **bayan** bundle.)

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5 levels, increasing speed / brick count) (✅ 0.3.0, 2026-05-26)
- M3 — 2.5D depth pass (parallax background, brick extrusion, debris + collapse) (✅ 0.4.0, 2026-05-26)
- M4 — audio pass (self-rolled square-wave SFX + mute; music deferred) (✅ 0.5.0, 2026-05-26; OSS sink → ALSA via vani-core at 0.8.0)
- M5 — high-score file (sankoch-compressed, sigil-HMAC-hashed, tamper-rejecting) (✅ 0.6.0, 2026-05-26)
- M6 — polish (menu, pause, framebuffer scaling, beveled art, accessibility) (✅ 0.7.0, 2026-05-26)
- v1.0 — end-to-end playable, polish complete ← next (after the console playtest gate; target 2026-06-13)

## Source

M6 complete — full game shell. Self-rolled on bare stdlib through M4 ([ADR 0003](../adr/0003-self-rolled-primitives.md)); M5 added the shared crates (sankoch + sigil); M6 is presentation polish over the existing systems.

Present (M1 loop + M2 levels + M3 depth + M4 audio + M5 scores + M6 polish — 382 assertions green):
- `src/main.cyr` — title menu (play / high scores / quit, `menu_loop`/`menu_move`) → `play_game` (real-time loop: tick → input → step → spawn-fx → SFX → render → fx → present, level advance, pause, mute) → high-score entry/table; headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english + speed-conserving `paddle_rebound_vx/vy` (discrete angle zones, ADR 0001), `hit_is_side` (which face of a brick was struck)
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion + `paddle_set_x`
- `src/bricks.cyr` — brick grid + destruction + scoring; per-cell *tier* (0=empty, 1..9), `bricks_from_cells`, `brick_tier`, `tier * value` scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` + `world_set_bricks` (level swap) + `world_speed`/`world_set_speed` (per-level target, paddle bounce re-asserts |v|) + `world_last_hit`/`world_last_tier` (debris hook) + `world_events` (SFX hook)
- `src/level.cyr` — 5 original ASCII layouts, plain-text parser, auto-fit geometry, `level_speed` per-level target (5.0 → 7.0) + serve curve, `level_make_bricks` + `level_load` (advance, score/lives carry)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()` over a parallax `render_bg()`; tier-coloured (`tier_rgb`) beveled bricks (`draw_brick`), rounded highlighted ball (`draw_ball`), lit-edge paddle (`draw_paddle`)
- `src/fx.cyr` — particle pool: debris shards + brick-collapse on destroy (gravity, life-shrink/dim); `fx_step`/`fx_render`
- `src/audio.cyr` — square-wave SFX synthesis (`synth_tone`, 6 SFX pre-rendered into a ~255 KB heap cache) in the device's own format, **48 kHz S16_LE stereo** (L == R; 11025 Hz U8 mono until 0.8.4) + WAV dump + mute; the tested, device-free core
- `src/sound.cyr` — best-effort ALSA SFX sink via vani-core, to `/dev/snd/pcmC{card}D{device}p` (card 1 device 0 by default, the `SoundDev` constants). `sound_open` sets `audio_set_params_fmt` S16_LE on an explicit 1024 × 32 = 32768-frame ring with silence-filled sw params (`sound_set_sw_silence`), falls back to a kernel-chosen ring, and closes a device that refuses the format (silent). `sound_play` calls `audio_write` and re-prepares + retries on XRUN (`sound_write_recoverable`); `sound_close` drains and closes. It replaced the OSS `/dev/dsp` sink at 0.8.0. The ioctl path is device-only; the retry policy and ring sizing are unit-tested
- `src/save.cyr` — high-score table: insert/qualify/serialize + `hs_encode`/`hs_decode` (sankoch zlib + sigil HMAC-SHA256, tamper-rejecting) + `initials_apply` + atomic `hs_save`/`hs_load` (`~/.cyrius-bb/scores.cyb`). Engine is buffer-pure + tested
- `src/hud.cyr` — score + lives overlay; 3×5 digit + A-Z font (`hud_draw_letter`/`hud_draw_char`/`hud_draw_text`)
- `src/input.cyr` — raw-tty input: one stateful decoder `input_feed` (arrows, reads split mid-sequence) behind `input_poll`, `input_nav` (menu) and `input_read_key` (initials); `bb_key_action` (a/d/arrows/space/q/m/p/Ctrl-C); `input_hold_dir` (decaying key-held latch so hold-to-move glides across key-repeat gaps; the opposite key drops it); `input_settle_step` (post-game screens wait for held keys to stop)
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit, probes geometry + integer-scales + centres, honours the pan offset, refuses a mode it can't draw (geometry helpers unit-tested; the blit itself is on-console only)
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

- `tests/cyrius-bb.fcyr` — fuzz harness over `hs_decode` and the input decoder (fixed seed, 38,000 cases, runs in CI as `cyrius fuzz`).
- `tests/cyrius-bb.tcyr` — **382 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios + collision events, framebuf, render + beveled-art/parallax, fx particle pool, audio synth (48 kHz S16_LE values, byte order, L == R, amplitude bound, frame offsets, cue lengths) + mute, the sound shell's pure pieces (ring sizing vs the worst one-tick burst, XRUN-retry policy, no-device / muted no-op), hud digits + letters, input decode, serve, level parser/tier-scoring/speed-curve, clear→advance progression, high-score qualify/insert/serialize/encode-decode/tamper). Deterministic + headless. The suite includes `vendor/vani-core.cyr` + `src/sound.cyr` but never opens a device.
- Playtest gate — the full interactive flow (menu → play → pause → game-over → high-score entry) + `/dev/fb0` present (now resolution-probing — verify scale/centre on the real console) + audible ALSA (`/dev/snd`, via vani-core; device path hardware-probed 2026-09-26, listening pending) + the *feel* of depth/audio/speed (parallax rate, palette, debris spread, SFX rhythm, deferred camera shake) need a real Linux console + audio device. Build/lint + headless-smoke + frame/WAV-dump-verified only so far; logic gated by 382 assertions + the `<frames>` smoke, save engine by an out-of-band disk round-trip. `present.cyr`'s ioctl probe is the highest-risk untested path.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input, audio synth — on bare stdlib. (Audio *playback* moved to the vendored vani-core ALSA shim at 0.8.0, replacing the OSS `/dev/dsp` sink; the synth stays self-rolled.)
- **Save / scoring** (WIRED at M5): sankoch (compression) + sigil (HMAC hash), + transitive freelist/bayan/ct/keccak/thread_local/random/sys. All are in `[deps].stdlib` and resolve from the 6.7.18 toolchain snapshot, with no git fetch (`bayan` is the carved-out former `bigint`). They are the bulk of the 1,165,272 B DCE binary ([note 002](../architecture/002-save-deps-binary-size.md)).
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing), shravan (audio — self-rolled instead, see [note 001](../architecture/001-no-ffi-audio.md)) — all commented out in `cyrius.cyml`. The build's only warning is the static-data advisory.

## Next

Start here next session (0.9.2 is the current cut — all milestones M0–M6 done):

1. **Console playtest** (THE pre-v1.0 gate — user-driven, planned for the morning after 0.7.0). On a real Linux console, run the whole flow: menu → play 5 levels → pause → lose all lives / clear all → high-score entry → table → back to menu. Verify `/dev/fb0` present scales + centres correctly at the console's resolution (the highest-risk untested code — `present.cyr` ioctl probe); audible ALSA via vani-core (card/device are the `SoundDev` constants in `src/sound.cyr`; [note 001](../architecture/001-no-ffi-audio.md)). Before judging audio, log in on the seat and make sure the codec's mixer is up and nothing else holds the card (note 001's last two gotchas). Then listen for all six cues: paddle blip, brick chirp, wall thud, ball-lost blip, game-over sting, level-clear fanfare. Tune *feel*: brick side hits (new at 0.9.2; a bot's clears ran about a third longer, and the audit report says how to revert), ball speed curve, paddle english, parallax rate, palette, debris spread, SFX rhythm; decide camera shake (deferred from M3) + a voice mixer if SFX overlap badly. Capture issues → fix.
2. **v1.0 close** — once the playtest is clean: closeout pass (CLAUDE.md), final size note, CI matrix green, then cut v1.0 (target **2026-06-13**, comfortably ahead). Knife article outline at [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md).
3. **Binary size** (optional) — 1,165,272 B DCE at 0.9.2 ([note 002](../architecture/002-save-deps-binary-size.md)); revisit only if it becomes a release blocker. One consumer-side lever was measured at 0.8.3 and not applied. `src/save.cyr` could call `zlib_compress` / `zlib_decompress` directly instead of the `compress` / `decompress(FORMAT_ZLIB, …)` dispatchers, which lets DCE drop every other sankoch codec, including 2.8.0's Brotli decoder. That build is **778,680 B** (−22%), still passes 222/222, and writes byte-identical save files. Trimming sigil's share, and the ~348 KB bayan 1.11.1 added at 0.9.1, is still upstream work.
4. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).
5. **Audio sample format — resolved (0.8.4).** Found at 0.8.3 as a sign mismatch: the synth wrote *unsigned* 8-bit, and vani's `bits = 8` programs **S8**. The hardware probe then found a bigger problem. The ALC897 refuses 8-bit outright (`-EINVAL`, unchecked), so the game was silent, and the proposed `SND_PCM_FORMAT_U8` one-liner would not have helped. The fix renders 48 kHz S16_LE stereo, recovers the stream after each cue's XRUN, and silence-fills the ring (CHANGELOG `[0.8.4]`). The listening test is part of item 1.
6. **vani: busy-card open — resolved (0.8.4).** vani 1.2.6 fixed it upstream, and the re-vendor to 1.2.9 picks it up: a card held by another client now gives the null handle at once, so the game runs silent instead of hanging before its menu ([note 001](../architecture/001-no-ffi-audio.md)). Verified by vani's own hardware run, not re-run here (no device access at the bump), so confirm it during item 1.

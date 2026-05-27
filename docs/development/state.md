# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.5.0** — M4 (audio pass) complete. Era-spirit square-wave SFX synthesised on bare stdlib (no FFI, no shravan bundle — self-rolled like the renderer; see [architecture note 001](../architecture/001-no-ffi-audio.md)): paddle/brick/wall/ball-lost/game-over/level-complete SFX, best-effort OSS `/dev/dsp` sink, WAV dump for verification, mute toggle. Built on M1 primitives + M2 levels + M3 depth, bare stdlib ([ADR 0003](../adr/0003-self-rolled-primitives.md)). (Prior: 0.4.0 — M3 depth; 0.3.0 — M2 levels; 0.2.0 — M1 loop; 0.1.0 — scaffold.)

- **DCE binary**: 117,184 B (x86_64, static, stripped) — up from 111,688 B at 0.4.0 (+5,496 for synth + sink).
- **Tests**: 166 assertions, 0 failed (was 147). Lint + fmt clean. SFX WAV dumps validated (pcm_u8, 11025 Hz, mono, durations match).
- **Deps**: bare stdlib — zero external deps (sankoch + sigil re-wire at M5; shravan stays deferred — no consumable bundle).
- **Caveat**: interactive loop + `/dev/fb0` present + audible `/dev/dsp` playback + the 5-level playthrough + the *feel* of depth/audio are build/lint + headless-smoke + frame/WAV-dump-verified only (no console/framebuffer/audio device in dev/CI); logic proven via the `<frames>` smoke + 166 unit tests.

## Toolchain

- **Cyrius pin**: `6.0.1` (in `cyrius.cyml [package].cyrius`) — bumped from `5.7.11` 2026-05-25. All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5 levels, increasing speed / brick count) (✅ 0.3.0, 2026-05-26)
- M3 — 2.5D depth pass (parallax background, brick extrusion, debris + collapse) (✅ 0.4.0, 2026-05-26)
- M4 — audio pass (self-rolled square-wave SFX + OSS sink + mute; music deferred) (✅ 0.5.0, 2026-05-26)
- M5 — high-score file (sankoch-compressed, sigil-integrity-hashed) ← next
- v1.0 — end-to-end playable, polish complete

## Source

M4 complete — the game has sound. Per [ADR 0003](../adr/0003-self-rolled-primitives.md), cyrius-bb tools its own primitives on bare stdlib (cyrius-doom pattern) rather than waiting on kiran/impetus/mabda/shravan.

Present (M1 loop + M2 progression + M3 depth + M4 audio — 166 assertions green):
- `src/main.cyr` — real-time loop (tick → input → step → spawn-fx → SFX → render → fx → present), level-index tracking (advance on clear, game-complete on final clear), mute toggle + headless `<frames>` smoke
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
- `src/hud.cyr` — score + lives overlay (3×5 bitmap digit font)
- `src/input.cyr` — keyboard raw-tty input (a/d/arrows/space/q/m) + pure `bb_key_action` decoder
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit (untested in CI; on-console only)
- `programs/demo.cyr` — frame eyeball harness (`build/frame00..02.ppm`); `programs/audio_demo.cyr` — SFX ear harness (`build/sfx_*.wav`)

Planned modules:
- `src/save.cyr` — high-score file I/O (sankoch + sigil) (M5)

## Assets

Scaffold — no assets shipped yet.

Planned per [ADR 0002](../adr/0002-original-assets-only.md):
- Original sprite set (paddle, ball, brick variants) — new pixel art
- Era-spirit palette landed in M3 (`tier_rgb` — original 9-tier cool→warm band, not Atari's row colours)
- Original square-wave SFX landed in M4 (`src/audio.cyr` — self-rolled, no shravan; see [note 001](../architecture/001-no-ffi-audio.md))
- 5 original level layouts landed in M2 (`src/level.cyr` — ASCII grids, ADR 0002); final art pass in M6

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — **166 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios + collision events, framebuf, render + tier colour/extrusion, parallax bg, fx particle pool, audio synth/mute, hud, input decode, serve, level parser/tier-scoring/speed-curve, clear→advance progression). Deterministic + headless.
- Playtest gate — every milestone requires manual playthrough (feel concerns). The interactive loop + `/dev/fb0` present + audible `/dev/dsp` playback + the 5-level playthrough + the *feel* of the depth/audio effects (parallax rate, palette, debris spread, SFX rhythm, deferred camera shake) still need a real Linux console + audio device (build/lint + headless-smoke + frame/WAV-dump-verified only so far); headless logic is gated by the 166 assertions + the `<frames>` smoke.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input, **audio synth + OSS sink** — on bare stdlib
- **Save / scoring**: sankoch (compression), sigil (hash) — commented out in `cyrius.cyml`, re-wired at M5
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing), **shravan (audio — no consumable bundle; self-rolled instead, see [note 001](../architecture/001-no-ffi-audio.md))** — all commented out in `cyrius.cyml`. The build is now bare-stdlib, warning-free.

## Next

Start here next session (0.5.0 is the current cut):

1. **M5 — high-score persistence** (v0.6.0). `src/save.cyr` — top-10 scores with initials saved to `~/.cyrius-bb/scores.cyb`, sankoch-compressed + sigil-integrity-hashed; score-entry UI on a qualifying game-over; tamper-detection (hash mismatch → refuse to load, don't crash). This is where sankoch + sigil get re-wired — uncomment their `[deps.*]` blocks in `cyrius.cyml` (tags 2.1.0 / 2.9.3 are available) and `cyrius deps`. Acceptance: scores persist across sessions; hand-editing the file invalidates it; a legit clear produces a hand-calculable score. See [`roadmap.md`](roadmap.md) M5.
2. **Console + device playtest** (carries from M1–M4) — verify the interactive loop + `/dev/fb0` present + audible `/dev/dsp` audio (likely needs `padsp ./cyrius-bb` on a modern box; see [note 001](../architecture/001-no-ffi-audio.md)) + the full 5-level playthrough on a real Linux console. Tune the *feel*: depth effects (parallax rate, palette, debris spread), SFX rhythm + a possible voice mixer, camera shake (deferred from M3). Add a game-over / game-complete screen (M6). Fix any blit/audio format issues.
3. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).
4. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone — captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

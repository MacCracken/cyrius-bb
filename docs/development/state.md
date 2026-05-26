# cyrius-bb — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures (durable); this file is **state** (volatile).

## Version

**0.2.0** — M1 (foundational loop) complete. Playable brick-breaker on self-rolled primitives + bare stdlib ([ADR 0003](../adr/0003-self-rolled-primitives.md)): fixed-point physics, offscreen renderer, HUD, raw-tty input, real-time loop. (Prior: 0.1.0 — scaffold + primitives, 2026-05-25.)

- **DCE binary**: 98,648 B (x86_64, static, stripped) — down from 748,032 B at 0.1.0 after dropping the dormant mabda/sankoch/sigil deps.
- **Tests**: 93 assertions, 0 failed. Lint + fmt clean.
- **Deps**: bare stdlib — zero external deps (sankoch + sigil re-wire at M5).
- **Caveat**: interactive loop + `/dev/fb0` present are build/lint-verified only (no console/framebuffer in dev/CI); loop logic proven via the headless `<frames>` smoke + unit tests.

## Toolchain

- **Cyrius pin**: `6.0.1` (in `cyrius.cyml [package].cyrius`) — bumped from `5.7.11` 2026-05-25. All 16 stdlib modules in the dep list resolve against 6.0.1; no manifest changes beyond the pin.

## Milestones

See [`roadmap.md`](roadmap.md). Immediate sequence:

- M0 — scaffold (✅ 2026-04-24)
- M1 — ball + paddle + bricks, collision, render loop, HUD, input (✅ 0.2.0, 2026-05-25)
- M2 — level system (5+ levels, increasing speed / brick count) ← next
- M3 — 2.5D depth pass (parallax background, brick-destruction perspective, particle debris)
- M4 — audio pass (shravan-synthesized effects, optional slot-loaded music)
- M5 — high-score file (sankoch-compressed, sigil-integrity-hashed)
- v1.0 — end-to-end playable, polish complete

## Source

M1 complete — the full playable loop is in place. Per [ADR 0003](../adr/0003-self-rolled-primitives.md), cyrius-bb tools its own primitives on bare stdlib (cyrius-doom pattern) rather than waiting on kiran/impetus/mabda.

Present (M1 playable loop — 93 assertions green):
- `src/main.cyr` — real-time loop (tick → input → step → render → present) + headless `<frames>` smoke
- `src/fixed.cyr` — 16.16 fixed-point math (deterministic, integer-only)
- `src/geom.cyr` — AABB collision, axis-aligned reflection, paddle english
- `src/ball.cyr` — ball entity + velocity integration
- `src/paddle.cyr` — paddle entity + bounded horizontal motion
- `src/bricks.cyr` — brick grid + destruction + scoring
- `src/world.cyr` — `world_step()` tick + `world_serve` (re-serve after loss)
- `src/framebuf.cyr` — offscreen RGB surface + clipped `fb_fill_rect` + PPM dump (self-rolled)
- `src/render.cyr` — `render_world()`: bricks/paddle/ball as flat rects
- `src/hud.cyr` — score + lives overlay (3×5 bitmap digit font)
- `src/input.cyr` — keyboard raw-tty input (a/d/arrows/space/q) + pure `bb_key_action` decoder
- `src/tick.cyr` — ~60 fps frame pacing
- `src/present.cyr` — best-effort `/dev/fb0` blit (untested in CI; on-console only)
- `programs/demo.cyr` — eyeball harness; dumps `build/frame00..02.ppm`

Planned modules:
- `src/level.cyr` — level loading, progression, speed-curve (M2)
- `src/audio.cyr` — sound-effect synthesis (M4)
- `src/save.cyr` — high-score file I/O (sankoch + sigil) (M5)

## Assets

Scaffold — no assets shipped yet.

Planned per [ADR 0002](../adr/0002-original-assets-only.md):
- Original sprite set (paddle, ball, brick variants) — new pixel art
- Era-spirit palette (not pixel-matching Atari Breakout's specific row colors)
- Original square-wave / FM sound effects via shravan
- Placeholder layouts in M1; 5+ original level layouts land in M2; final art pass in M6

No Atari-era assets. No ROM extraction. No ML-generated derivatives of Atari art.

## Tests

- `tests/cyrius-bb.tcyr` — **93 assertions**, 0 failed (fixed-point, geometry, ball, paddle, bricks, `world_step` scenarios, framebuf, render, hud, input decode, serve). Deterministic + headless.
- Playtest gate — every milestone requires manual playthrough (feel concerns). The interactive loop + `/dev/fb0` present still need a real Linux console to actually playtest (build/lint-verified only so far); headless logic is gated by the 93 assertions + the `<frames>` smoke.

## Dependencies

Per [ADR 0003](../adr/0003-self-rolled-primitives.md):
- **Core stdlib**: syscalls, alloc, fmt, io, fs, str, string, vec, args, hashmap, process, thread, fnptr, chrono, tagged, assert
- **Self-rolled** (no dep): fixed-point math, collision, game loop, framebuffer render, input — on bare stdlib
- **Save / scoring**: sankoch (compression), sigil (hash) — commented out in `cyrius.cyml`, re-wired at M5
- **Deferred** (later in-depth projects, not this one): kiran (engine), impetus (physics), mabda (GPU), soorat (windowing) — all commented out in `cyrius.cyml`. The build is now bare-stdlib, warning-free.

## Next

Start here next session (0.2.0 is tagged):

1. **M2 — level progression** (`src/level.cyr`, v0.3.0). Load level layouts from a data file, advance on clear, speed-curve + brick-count scaling across levels, lives carried across levels, 5+ original layouts (per ADR 0002). The single-level `world` already exists — M2 turns it into a sequence. See [`roadmap.md`](roadmap.md) M2 for acceptance.
2. **v0.2.x patch** — verify the interactive loop + `/dev/fb0` present on a real Linux console (the one M1 surface not verifiable in dev/CI). Fix any blit format / resolution issues found.
3. **Repo hygiene** (optional, anytime) — untrack the vendored stdlib: `git rm -r --cached lib/` + add `lib/*.cyr` to `.gitignore` (per first-party standards; CI regenerates via `cyrius deps`).
4. Knife article for cyrius-bb is outlined at fold-complete / v1.0 milestone — captured in [agnosticos/docs/articles/_outlines.md](../../../../Repos/agnosticos/docs/articles/_outlines.md) at the appropriate time.

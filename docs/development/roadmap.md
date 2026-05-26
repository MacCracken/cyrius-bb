# cyrius-bb — Roadmap

> Milestone plan through v1.0. State lives in [`state.md`](state.md); this file is the sequencing.

## Guiding objective

**Ship the simplest complete brick-breaker the Cyrius game stack can carry.** Deliberately scope-restrictive — the point is stack demonstration at accessible complexity, not engineering-novelty. If a feature requires a technical bet on par with cyrius-braid's time-rewind ring buffer, it's out of scope.

## Milestones

### M0 — Scaffold (v0.1.0) — ✅ shipped 2026-04-24

- `cyrius init cyrius-bb` + library-vs-binary decision (binary — game, not library)
- ADR 0001 (homage-from-observation thesis) + ADR 0002 (original-assets-only policy)
- Docs scaffolded per [first-party-documentation.md](https://github.com/MacCracken/agnosticos/blob/main/docs/development/first-party/first-party-documentation.md)

### M1 — Foundational loop (v0.2.0) — ✅ shipped 2026-05-25

*The "it's a game" milestone. Without M1, everything else is impossible.*

Built on self-rolled primitives, not kiran/impetus/mabda — see [ADR 0003](../adr/0003-self-rolled-primitives.md). 93 assertions green; DCE binary 98,648 B. Interactive loop + `/dev/fb0` present are build/lint-verified only (no console in dev/CI).

Done:
- ✅ `src/fixed.cyr` — 16.16 fixed-point math
- ✅ `src/geom.cyr` — AABB collision, reflection, paddle english
- ✅ `src/ball.cyr` — ball entity + velocity integration
- ✅ `src/paddle.cyr` — bounded horizontal motion (input wiring pending)
- ✅ `src/bricks.cyr` — brick grid, destruction-on-hit, scoring
- ✅ `src/world.cyr` — `world_step()`: integrate → wall/paddle/brick collision → score/lives/state

Done (cont.):
- ✅ `src/framebuf.cyr` — offscreen RGB surface + clipped fill-rect + PPM dump (self-rolled)
- ✅ `src/render.cyr` — `render_world()`: flat rects; depth treatment deferred to M3
- ✅ `programs/demo.cyr` — eyeball harness dumping PPM frames

Done (cont.):
- ✅ `src/input.cyr` — raw-tty keyboard (a/d/arrows/space/q) + pure decoder
- ✅ `src/tick.cyr` — ~60 fps frame pacing
- ✅ `src/present.cyr` — best-effort `/dev/fb0` blit (on-console)
- ✅ `src/main.cyr` — real-time loop (tick → input → step → render → present) + headless `<frames>` smoke; mabda smoke test dropped, build now bare-stdlib
- ✅ `src/hud.cyr` — score + lives overlay (3×5 bitmap digit font)

Carried forward (not blocking M1 close):
- `/dev/fb0` present verification on a real Linux console (untestable under this desktop session)
- Placeholder assets stay solid-color rectangles until the art pass (M6)

**Acceptance** (met): a single-level game is playable end-to-end — ball bounces off walls + paddle + bricks; bricks destruct on hit; score increments; all-bricks-destroyed fires the level-complete state. Verified via 93 headless assertions + the `<frames>` smoke.

### M2 — Level progression (v0.3.0)

*Turn the single level into a game with depth.*

- `src/level.cyr` — level loader from data file, progression on clear, speed-curve across levels
- 5+ original level layouts (not Atari-identical; see ADR 0002)
- Lives system — 3 lives, game-over on zero
- Score accumulation across levels
- Ball-speed + brick-count scaling across levels

**Acceptance**: player can play through 5 levels with increasing difficulty, lose all lives, see final score. Level data loadable from a simple format (CYML or plain text lines).

### M3 — 2.5D depth pass (v0.4.0)

*Visual treatment that makes the "2.5D" claim real.*

- Parallax background — 2-3 layers of depth-offset scrolling when the ball moves
- Brick-destruction perspective — bricks "tilt" slightly as they destruct (not a rotation animation, just a Z-depth illusion)
- Particle debris on brick destruction — colored shards that drift briefly
- Subtle camera shake on impacts (optional; playtest-gated)

**Acceptance**: the game reads as "2.5D arcade" vs "flat 2D" — a player with no context could tell the difference from a screenshot.

### M4 — Audio pass (v0.5.0)

- `src/audio.cyr` — sound effects via shravan: paddle-ball blip, brick-destroy chirp, wall-bounce thud, game-over sting, level-complete fanfare
- Era-spirit synthesis: square wave / simple FM via shravan primitives, no sampled audio
- Optional slot-loaded music (user-provided `.ogg` files at `~/.cyrius-bb/music/<level>.ogg`); silent-by-default if absent
- Mute toggle

**Acceptance**: audio adds to the feel without demanding attention. Playtest confirms the SFX rhythm matches the gameplay rhythm.

### M5 — High-score persistence (v0.6.0)

- `src/save.cyr` — high-score table saved to `~/.cyrius-bb/scores.cyb` (custom extension, sankoch-compressed, sigil-integrity-hashed)
- Top-10 persistent scores with player initials
- Score entry UI on game-over if the score qualifies
- Tamper-detection — sigil hash mismatch means corrupted save; refuse to load, don't crash

**Acceptance**: high scores persist across sessions; editing the file by hand invalidates it; a legit clear path produces a score that matches a hand-calculated expectation.

### M6 — Polish (v0.9.0)

- Menu screen (title, play, high scores, quit)
- Pause functionality
- Screen-size / windowed-mode handling
- Final art pass — any placeholder assets replaced with their final original versions per ADR 0002
- Accessibility pass — keyboard-only play, color-blind-friendly brick colors

### v1.0 — Ship (**2026-06-13 — Saturday**)

**Target date pinned 2026-04-24.** The logic:

- **Breakout original US arcade release**: April 13, 1976 (most-cited date; subject to primary-source verification — see [`../design/breakout-date-verification.md`](../design/breakout-date-verification.md) for the source-checking action item).
- **cyrius-bb ship date**: **2026-06-13, Saturday**. That's 50 years + 2 months after the Breakout reference date, same day-of-month (13). Weekend = indie-game-audience attention. Echoes the original without claiming to BE the literal anniversary (which was 2026-04-13 and has passed).
- **Relationship to the summer-2026 arc**: ships **8 days before the Beat 1 solstice demo (2026-06-21)**. cyrius-bb gets its own release moment; Beat 1 then pulls it into the broader demo context alongside cyrius-braid as a matched pair (accessibility + depth).
- **Calendar**: 50 days from scaffold (2026-04-24) to ship (2026-06-13). M1–M6 need to land at ~8 days per milestone. Aggressive but feasible given the deliberately-modest scope.

**Scope at v1.0**:
- All M1–M6 complete, polish pass done
- Playtest against documented Breakout-era mechanics for feel
- CI matrix green — builds on all Cyrius-supported platforms
- Knife article outlined at [agnosticos/docs/articles/_outlines.md](https://github.com/MacCracken/agnosticos/blob/main/docs/articles/_outlines.md) — short-form piece on the 50-year homage + Cyrius game-stack proof

**Pre-release gate on the Breakout date verification**: if primary-source research (see the design dir note) turns up a different anniversary date than April 13, 1976, the June 13 ship date either (a) still holds as "50 years + 2 months narrative framing regardless of exact day" or (b) shifts to align with a newly-verified anniversary day — decision made once the source check completes, not speculatively now.

## Post-v1.0 (not scheduled)

- **Additional levels** — the original six-world Breakout scope was small; community level contributions via CC-BY welcome
- **Multi-ball power-ups** — documented in some Breakout-era variants; falls out naturally from the ball abstraction
- **Speedrun mode** — deterministic ball physics make frame-perfect runs practical
- **Level editor** — CC-BY community levels; needs file-format versioning discipline first
- **Gamepad / rumble support** — better than keyboard-only for the genre

## Why this scope

cyrius-bb is deliberately modest so it can ship. The retro-port lineage already has cyrius-doom (engine novelty), cyrius-braid (time-rewind novelty). cyrius-bb is the *"stack demo that anyone can pick up"* point on the lineage — proves the Cyrius game stack carries titles that don't need a novel engineering gate. Pair it with cyrius-braid in any stack demo: one is the proof-of-depth, the other is the proof-of-accessibility.

See [ADR 0001](../adr/0001-homage-from-observation.md) for the mechanics-from-observation thesis; [ADR 0002](../adr/0002-original-assets-only.md) for the assets policy; [`state.md`](state.md) for live progress.

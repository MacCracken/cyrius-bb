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

The "it's a game" milestone. Self-rolled on bare stdlib ([ADR 0003](../adr/0003-self-rolled-primitives.md)): fixed-point physics, AABB collision + paddle english, brick grid + scoring, offscreen renderer, HUD, raw-tty input, ~60 fps loop. Single level playable end-to-end (ball bounces off walls/paddle/bricks; bricks destruct + score; all-cleared fires level-complete). 93 headless assertions; DCE binary 98,648 B. Per-module detail in [`CHANGELOG.md`](../../CHANGELOG.md) `[0.2.0]`.

**Carried forward** (not blocking; a v0.2.x patch or later):
- `/dev/fb0` present + interactive loop verified on a real Linux console — build/lint-verified only so far (no console/framebuffer in dev/CI).
- Placeholder assets stay solid-color rectangles until the art pass (M6).

### M2 — Level progression (v0.3.0) — ✅ shipped 2026-05-26

*Turn the single level into a game with depth.*

Shipped: `src/level.cyr` (plain-text layout parser + 5 original ASCII layouts, brick counts 18 → 25 → 34 → 40 → 49), advance-on-clear, per-level speed curve (vy −4 → −6 px/tick), score + lives carried across levels, game-over on zero lives. Per-cell brick *tier* (0–9) scores `tier × value` and seeds M3's per-tier colour. 127 headless assertions; DCE binary 103,400 B. Detail in [`CHANGELOG.md`](../../CHANGELOG.md) `[0.3.0]`.

**Carried forward** (not blocking; same gate as M1's loop):
- Interactive 5-level playthrough + game-over / game-complete presentation on a real Linux console — headless-smoke + unit-test-verified only so far. A dedicated game-over / win screen is M6 polish; the loop currently just ends on those states.

- `src/level.cyr` — level loader from data file, progression on clear, speed-curve across levels
- 5+ original level layouts (not Atari-identical; see ADR 0002)
- Lives system — 3 lives, game-over on zero
- Score accumulation across levels
- Ball-speed + brick-count scaling across levels

**Acceptance**: player can play through 5 levels with increasing difficulty, lose all lives, see final score. Level data loadable from a simple format (CYML or plain text lines).

### M3 — 2.5D depth pass (v0.4.0) — ✅ shipped 2026-05-26

*Visual treatment that makes the "2.5D" claim real.*

Shipped: parallax background (`render_bg` — depth zones + far/near vertical-bar layers shifting with ball x at different rates), per-tier brick colour (`tier_rgb` — original 9-tier band, ADR 0002) with a 2px extrusion shadow (raised-block read), and brick-destruction debris + collapse particles (`src/fx.cyr` — deterministic fixed-point pool, gravity + life-shrink/dim). 147 headless assertions; DCE binary 111,688 B. Frame dumps confirm the 2.5D read. Detail in [`CHANGELOG.md`](../../CHANGELOG.md) `[0.4.0]`.

**Carried forward** (not blocking):
- Camera shake on impact (the optional bullet below) deferred to the console-playtest pass — not screenshot-visible, best tuned live.
- Feel of the depth effects (parallax rate, palette, debris spread) wants a console eye.

- Parallax background — 2-3 layers of depth-offset scrolling when the ball moves
- Brick-destruction perspective — bricks "tilt" slightly as they destruct (not a rotation animation, just a Z-depth illusion)
- Particle debris on brick destruction — colored shards that drift briefly
- Subtle camera shake on impacts (optional; playtest-gated)

**Acceptance**: the game reads as "2.5D arcade" vs "flat 2D" — a player with no context could tell the difference from a screenshot.

### M4 — Audio pass (v0.5.0) — ✅ shipped 2026-05-26

- `src/audio.cyr` — sound effects ~~via shravan~~ **self-rolled** (shravan has no consumable bundle + ALSA/Pulse need FFI; see [note 001](../architecture/001-no-ffi-audio.md)): paddle-ball blip, brick-destroy chirp, wall-bounce thud, ball-lost blip, game-over sting, level-complete arpeggio
- Era-spirit synthesis: square wave (`synth_tone`, freq + amplitude sweep), unsigned 8-bit mono PCM — no sampled audio. Faithful to Breakout's 1976 square beeps
- `src/sound.cyr` — best-effort OSS `/dev/dsp` sink; `audio_write_wav` + `programs/audio_demo.cyr` make the SFX hearable without a device
- Mute toggle ('m')

166 headless assertions; DCE binary 117,184 B. Detail in [`CHANGELOG.md`](../../CHANGELOG.md) `[0.5.0]`.

**Acceptance**: audio adds to the feel without demanding attention. Playtest confirms the SFX rhythm matches the gameplay rhythm.

**Carried forward** (not blocking):
- **Slot-loaded music deferred** — `.ogg` needs an OGG decoder, infeasible without FFI/a decoder crate. If revisited it will be WAV. The SFX-focused acceptance does not depend on music.
- **Audible playback** needs an OSS device (`padsp` shim on modern desktops) — build-verified only, like `/dev/fb0`. WAV dumps are the always-works verification path. SFX *feel* + a possible voice mixer want a console ear.

### M5 — High-score persistence (v0.6.0) — ✅ shipped 2026-05-26

- `src/save.cyr` — high-score table at `~/.cyrius-bb/scores.cyb` (`CYBB` container, sankoch zlib-compressed, sigil HMAC-SHA256-hashed over the plaintext)
- Top-10 persistent scores with player initials (A-Z font added to `hud.cyr`)
- Score-entry screen on a qualifying natural finish + a high-score table view (console-side in `main.cyr`)
- Tamper-detection — HMAC mismatch (-2) or bad magic/size (-1) → refuse, leave the in-memory table untouched, don't crash

First milestone to USE the shared crates (ADR 0003): sankoch + sigil wired from the 6.0.1 toolchain snapshot via `[deps].stdlib`. 196 headless assertions + an out-of-band disk round-trip; DCE binary **454,352 B** (~4x — the crate cost, [note 002](../architecture/002-save-deps-binary-size.md)). Detail in [`CHANGELOG.md`](../../CHANGELOG.md) `[0.6.0]`.

**Acceptance**: high scores persist across sessions; editing the file by hand invalidates it; a legit clear path produces a score that matches a hand-calculated expectation. ✅ All verified (in-memory tests + disk round-trip; scores are `tier × 10` summed, hand-calculable).

**Carried forward** (not blocking):
- The score-entry + high-score screens are first-pass console UI (untested in CI, like the loop); their layout/feel + the game-over→menu flow land in M6.
- Binary size (~4x) accepted as the honest cost of real compression + crypto.

### M6 — Polish (v0.7.0) — ✅ shipped 2026-05-26

> Cut as v0.7.0 (not the originally-pinned v0.9.0) — the 0.7/0.8 headroom
> was unneeded once M1–M5 landed fast; v1.0 still targets 2026-06-13.

- ✅ Menu screen (title / play / high scores / quit) — `menu_loop` in `main.cyr`, keyboard nav
- ✅ Pause ('p') — sim frozen + PAUSED overlay
- ✅ Screen-size handling — `present.cyr` probes the framebuffer geometry and integer-scales + centres (no windowed mode: it's a `/dev/fb0` console target, not a window system)
- ✅ Final art pass — procedural, per ADR 0002 (no asset files): beveled bricks, rounded highlighted ball, lit-edge paddle (`render.cyr`)
- ✅ Accessibility — keyboard-only play + menus; bevel shape-cue + row-position disambiguate tiers beyond hue (full CVD repalette left as a playtest tweak)

199 headless assertions; DCE binary **460,384 B**. Detail in [`CHANGELOG.md`](../../CHANGELOG.md) `[0.7.0]`.

**Carried forward** (the pre-v1.0 gate): full console playthrough of the flow + `/dev/fb0` scale/centre verification (the riskiest untested path) + audible `/dev/dsp` + feel tuning + camera-shake decision. See `state.md` Next.

### Game-loop review backlog (target 0.7.2 — observed on first console playtest)

> Two feel/physics issues surfaced on the first real-VT playthrough (after the
> 0.7.1 input fix). **Both resolved in 0.7.2** (deterministic + unit-tested
> cores); the analysis is kept below for the record. They are "feel" bugs whose
> exact numbers still want a console eye to tune (CLAUDE.md: *playtest over unit
> test for feel*) — the *mechanism* is fixed, the *tuning* is a playtest knob.

**✅ 1. Paddle input feels slowed — hold-to-move isn't continuous.** *(0.7.2 —
fixed via direction (a): the decaying key-held latch, `input_hold_step` /
`PADDLE_HOLD_FRAMES`. Latch length is the playtest knob; if the one-time
initial-repeat stall still bites, layer direction (b) `KDKBDREP` on the console.)*
- *Symptom*: holding `a`/`d` (or arrows) lurches the paddle — a step, a stall, then steppy motion — instead of a smooth glide.
- *Root cause (hypothesis)*: the loop treats "a movement byte arrived this frame" as "key held this frame" (`input_poll` → `ACT_LEFT`/`ACT_RIGHT` → one `paddle_move` of `dx=4px`). But raw-tty delivers the OS **key-repeat stream**, not a held state: ~250 ms initial-repeat delay, then ~30 bytes/s. At 60 fps that's one move, a ~15-frame stall, then a move every ~2 frames — exactly the "slowed, not respecting hold-down" feel.
- *Fix directions (decide on review)*: **(a)** sticky/decaying key-held latch in the input layer — a movement byte arms "held for N frames", refreshed by each repeat byte, so the paddle moves every frame across the repeat gaps (tune N just over the repeat interval); smallest change, fits the existing poll model. **(b)** set the console repeat rate via the `KDKBDREP` tty ioctl (shorter delay/faster rate) — global console state, blunter. **(c)** read `/dev/input/eventX` for true key down/up — most correct, heaviest; weigh against accessible scope.

**✅ 2. Ball speed changes for no physical reason (appears to speed up randomly).**
*(0.7.2 — fixed: `paddle_rebound_vx/vy` give a speed-conserving rebound — english
selects a discrete angle zone, the ball leaves at the level's target speed.
Added `level_speed` (5.0 → 7.0) as the per-level scaling so speed is constant
within a level and steps up between levels. Per-level numbers are a playtest knob.)*
- *Symptom*: total ball speed jumps between rallies with no visible cause.
- *Root cause*: the paddle bounce **does not conserve speed**. `world_step`'s paddle branch *sets* `vx = english × WO_ENGLISH` (english ∈ [−1,1], max 5 px/tick) and independently *negates* `vy` (magnitude unchanged). Outgoing speed = √(vy² + vx²): a centre hit (vx≈0) keeps speed, an edge hit (vx≈±5) **adds** a horizontal component without removing vertical → the ball gets faster. Since `vy` magnitude is otherwise invariant for the whole level (walls/bricks/paddle only ever negate a component), *where the ball strikes the paddle* is the only thing that moves total speed — which reads as inexplicable speed-up.
- *Correct behavior*: paddle english sets the outbound **angle** at **constant speed**. Fix direction: on a paddle hit, take speed = |v_in| (or the level's target speed), derive an outbound angle from the english offset, then set (vx, vy) = speed·(sinθ, −cosθ) so |v_out| is preserved. Keep the per-level speed curve as the *only* intentional speed source.
- *Related (⏳ still open — not done in 0.7.2)*: brick collisions always `reflect(vy)` regardless of which side was struck (`world_step` brick branch) — side hits should flip `vx`. Not a speed bug, but the same axis-naive collision-response family. And resolution is single-axis-per-frame with no swept test, so tunneling/double-hit edge cases may appear at higher speeds (the `level_speed` cap of 7 < brick height 8 keeps the current ramp safe; revisit if speeds rise). Carry to a later pass.

*Acceptance for the review pass*: holding a direction glides the paddle smoothly from the first frame; ball speed is constant within a level except the intended per-level step-up; edge-vs-centre paddle hits change **angle**, not speed. — **Mechanism met in 0.7.2** (unit-tested); smooth-glide *feel* + exact numbers confirmed on the console playtest.

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

# cyrius-bb

> *Break-breaker* — 2.5D brick-breaker in Cyrius. 50-year homage to *Breakout* (Atari, 1976).
> A **LIGHT\*OCEAN\*STUDIOS** production.

---

## What this is

`cyrius-bb` (*break-breaker*) is a 2.5D brick-breaker game reimplemented from observation, in Cyrius. 50-year anniversary homage to [*Breakout*](https://en.wikipedia.org/wiki/Breakout_(video_game)) — Atari, 1976 — the seminal title in the genre.

Same shape as [cyrius-doom](https://github.com/MacCracken/cyrius-doom), [cyrius-nba-jam](https://github.com/MacCracken/cyrius-nba-jam), [encom-hits](https://github.com/MacCracken/encom-hits), and [cyrius-braid](https://github.com/MacCracken/cyrius-braid) — retro-game-port-in-Cyrius, proving the Cyrius game stack carries accessible arcade titles. Deliberately the simplest technical bet in the retro-port series: no time-rewind buffer, no novel engineering gate. Ship in a week.

## What this isn't

- **Not a direct port of Atari Breakout.** Original source + assets aren't public; this is structural fidelity to the documented mechanics, not byte-equivalence.
- **Not using Atari-era assets.** All art, sound, and level design original per ADR 0002. No ripping, no "inspired-by" substitutes pretending to be the original.
- **Not a raycaster pseudo-3D.** The "2.5D" here means orthogonal 2D with depth layering (parallax background, slight brick-destruction perspective) — not a full 3D engine. Scope kept deliberately modest.

## Dependencies

**Bare Cyrius stdlib — zero external deps.** Per [ADR 0003](docs/adr/0003-self-rolled-primitives.md), cyrius-bb tools its own primitives (fixed-point math, collision, game loop, framebuffer, input) rather than depending on the heavier game stack, following the proven cyrius-doom pattern. An Atari-scope brick-breaker doesn't need an ECS engine or a rigid-body physics crate.

Two first-party crates are earmarked for later milestones (commented out in `cyrius.cyml` until then):

| Crate | Purpose | When |
|-------|---------|------|
| [sankoch](https://github.com/MacCracken/sankoch) | High-score file compression | M5 |
| [sigil](https://github.com/MacCracken/sigil) | High-score integrity hash | M5 |

Deferred to later, more in-depth projects (not this one): [mabda](https://github.com/MacCracken/mabda) (GPU), [kiran](https://github.com/MacCracken/kiran) (engine), [impetus](https://github.com/MacCracken/impetus) (physics).

No C. No FFI. Cyrius stdlib end to end.

## Build

```sh
cyrius deps
cyrius build src/main.cyr build/cyrius-bb
cyrius test tests/cyrius-bb.tcyr        # 84 assertions, headless + deterministic
./build/cyrius-bb 90                    # headless smoke: step 90 ticks, dump a PPM, print score
./build/cyrius-bb                       # interactive (Linux console — a/d move, q quits)
```

## Status

**0.1.0 released; M1 (playable loop) on the dev tip.** The full game runs headless and deterministic — ball/paddle/brick physics, paddle english, scoring, lives, level-clear/game-over — plus a self-rolled offscreen renderer (with a `/dev/fb0` present for console play) and raw-tty input. Built on bare stdlib per [ADR 0003](docs/adr/0003-self-rolled-primitives.md). Remaining M1 polish: a HUD and on-console `/dev/fb0` verification. See [`docs/development/roadmap.md`](docs/development/roadmap.md).

## License

GPL-3.0-only.

---

*Part of [AGNOS](https://github.com/MacCracken/agnosticos). See also: [cyrius-doom](https://github.com/MacCracken/cyrius-doom), [cyrius-nba-jam](https://github.com/MacCracken/cyrius-nba-jam), [cyrius-braid](https://github.com/MacCracken/cyrius-braid), [encom-hits](https://github.com/MacCracken/encom-hits).*

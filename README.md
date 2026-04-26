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

All first-party, all Cyrius-native (folded into stdlib):

| Subsystem | Purpose |
|-----------|---------|
| [mabda](https://github.com/MacCracken/mabda) | GPU rendering (2.5D sprite + parallax depth) |
| [kiran](https://github.com/MacCracken/kiran) | Game engine (ECS, scene hierarchy, input) |
| [impetus](https://github.com/MacCracken/impetus) | Physics (ball-brick-paddle collision, velocity curves) |
| [shravan](https://github.com/MacCracken/shravan) | Audio codec (sound effects, level music) |
| [hisab](https://github.com/MacCracken/hisab) | Vector / transform math |
| [sankoch](https://github.com/MacCracken/sankoch) | High-score file compression |
| [sigil](https://github.com/MacCracken/sigil) | High-score integrity hash |

No C. No FFI. Cyrius stdlib end to end.

## Build

```sh
cyrius deps
cyrius build src/main.cyr build/cyrius-bb
cyrius test src/test.cyr
```

## Status

**0.1.0 — scaffold.** Module skeletons + ADR 0001 (homage-from-observation) + ADR 0002 (original-assets policy) landed first. M1 (ball + paddle + single brick layer) is the foundation; the rest layers on. See [`docs/development/roadmap.md`](docs/development/roadmap.md).

## License

GPL-3.0-only.

---

*Part of [AGNOS](https://github.com/MacCracken/agnosticos). See also: [cyrius-doom](https://github.com/MacCracken/cyrius-doom), [cyrius-nba-jam](https://github.com/MacCracken/cyrius-nba-jam), [cyrius-braid](https://github.com/MacCracken/cyrius-braid), [encom-hits](https://github.com/MacCracken/encom-hits).*

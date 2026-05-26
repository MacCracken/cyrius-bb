# 0003 — Self-rolled primitives; defer kiran / impetus / mabda

**Status**: Accepted
**Date**: 2026-05-25

## Context

The original dep chain (see `cyrius.cyml` and `README.md`) assumed cyrius-bb
would stand on the full Cyrius game stack: **kiran** (ECS engine + scene
hierarchy + input), **impetus** (rigid-body physics), **mabda** (GPU
rendering), and a windowing layer (**soorat**). At toolchain 6.0.1, that stack
is not buildable here:

- **kiran** and **impetus** are still Rust — their Cyrius ports are pending,
  with `TBD` tags in the manifest. Neither repo exists locally.
- **soorat** (windowing) was never even wired into the dep chain (noted in
  `docs/development/tooling-pain-points.md` P6).
- **mabda** resolves, but it is GPU rendering only — no window, no input, no
  game loop. Rendering alone cannot make a playable game.

So M1 ("the foundational loop") as written is hard-blocked on upstream work
that has no schedule. Meanwhile **cyrius-doom** — a *shipped* retro port in
this same lineage — depends on none of them: it rolls its own framebuffer
(`/dev/fb0`), tick loop, fixed-point math, and input on bare stdlib (+ vani for
audio, bsp for level geometry). A brick-breaker is far simpler than DOOM: one
ball, one paddle, a brick grid, axis-aligned collision. The heavy stack is for
later, more in-depth projects, not an Atari-scope arcade title.

## Decision

**cyrius-bb tools its own primitives on bare Cyrius stdlib and does not depend
on kiran, impetus, mabda, or soorat.** Fixed-point math, geometry/collision,
the game loop, framebuffer rendering, and input are written in `src/` and
unit-tested headless — following the proven cyrius-doom pattern.

- **In scope now**: deterministic 16.16 fixed-point math (`src/fixed.cyr`),
  collision + reflection + paddle-english primitives (`src/geom.cyr`), and the
  entity/loop/render/input modules that build on them.
- **Retained deps**: **sankoch** (high-score compression) and **sigil**
  (high-score integrity hash) stay — they are M5 save-file concerns, not engine
  concerns, and have no self-roll justification (don't reinvent crypto/compression).
- **Deferred, not deleted**: kiran / impetus / mabda / soorat remain documented
  as the path for later in-depth projects. If a future cyrius-bb feature ever
  genuinely needs one, that's a new decision with its own ADR.

## Consequences

- **Positive** — unblocks the whole project immediately; keeps cyrius-bb at its
  intended "accessible scope / ship in a week" complexity; the simulation core
  is pure integer math, so it is deterministic and fully unit-testable without a
  window (speedrun-friendly, CI-friendly).
- **Positive** — no FFI, no GPU driver, no Rust-port dependency on the critical
  path. Matches the AGNOS sovereignty stance end to end.
- **Negative** — we own the rendering/input/loop code we'd otherwise have
  inherited. Mitigated by the small scope (cyrius-doom's framebuf is 145 lines,
  tick is 55) and by the fact that a brick-breaker needs a fraction of DOOM's.
- **Negative** — fixed-point instead of `mabda`/`impetus` float math means
  hand-managing precision. Mitigated: 16.16 is ample for screen-space coords.
- **Neutral** — `cyrius.cyml` and `README.md` dep tables, plus the M1 roadmap
  entry, need rewording away from the kiran/impetus/mabda framing. Tracked as a
  follow-up; this ADR is the rationale that change points back to.

## Alternatives considered

- **Wait for the ports.** Rejected — no schedule for kiran/impetus; blocks
  indefinitely against an aggressive v1.0 date (2026-06-13).
- **Use mabda for rendering, self-roll only the loop.** Rejected for the first
  cut — mabda still needs a windowing layer that isn't wired, and pulls GPU
  driver surface and the u256/keccak/fl_alloc link warnings for a game that
  draws flat rectangles. A `/dev/fb0` framebuffer is enough. mabda stays
  available if a real 2.5D depth pass (M3) later justifies it.
- **Float (f64) math instead of fixed-point.** Rejected — float risks
  cross-platform non-determinism, working against the homage-fidelity and
  speedrun goals. Integer 16.16 is bit-identical everywhere.

# 0001 — Homage from observation, not source port

**Status**: Accepted
**Date**: 2026-04-24

## Context

*Breakout* (Atari, 1976) is a canonical piece of game-design history. Nolan Bushnell and Steve Bristow designed the arcade cabinet; a young Steve Wozniak and Steve Jobs built the original implementation for Atari. The title has three available artifacts:

1. **The original ROMs** — commercial, covered by Atari's (now Warner-era, then Infogrames, then Atari SA) license.
2. **Public documentation** — 50 years of GDC talks, design writeups, postmortems, historical footage, academic game-studies papers, and gameplay videos. The mechanics are exhaustively discussed.
3. **Source code** — never publicly released.

A Cyrius rebuild has three viable paths:

- **Source port** — source isn't public. Not available.
- **ROM disassembly / reverse engineering** — technically possible, legally ambiguous, culturally wrong for this project.
- **Reimplementation from observation** — rebuild mechanics from the public documentation trail. The same path any external researcher uses to write about Breakout's design.

Same shape as [cyrius-doom](https://github.com/MacCracken/cyrius-doom) (Carmack's engine, public documentation trail) and [cyrius-braid](https://github.com/MacCracken/cyrius-braid) (Blow's game, public documentation trail). Pattern established; cyrius-bb applies it.

## Decision

**cyrius-bb is a reimplementation from observation.** Every mechanic traces to a documented public source (interview, historical postmortem, gameplay footage timestamp, academic writeup). The original Atari Breakout ROM is never disassembled, emulated-and-instrumented, or fed to any tool — including ML-based analysis. The rebuild works from the same artifacts any historian has access to.

Visual style is a homage to the era's aesthetic (bold primaries, monospace score, block-pixel sprites) without copying Atari's specific design. Audio is new work in the spirit of early-arcade sound chips (square-wave blips, simple FM synthesis), not samples or re-recordings.

Scope of this decision:

- **In**: documented mechanics (ball wall-bounce angle physics, paddle english, brick-destruction rules, progressive speed increase per level, the canonical "narrower-paddle-after-breakthrough" rule, two-ball power-ups if documented).
- **Out**: ROM disassembly; sprite or sound extraction; any LLM-assisted binary analysis; level geometry harvested from emulator frame dumps.

## Consequences

- **Positive**
  - Preserves commercial respect for the original title + Atari SA's current license.
  - Preserves AGNOS's sovereignty stance — same stance that refuses C dependencies refuses analyze-the-incumbent-binary shortcuts.
  - Produces a legitimately independent implementation. Every feel-tuning decision lands through playtest, not by matching a reference emulator.
  - Keeps the door open for original creative direction — we don't have to pixel-match 1976 graphics.
- **Negative**
  - Feel-fidelity is harder without a reference ROM to match against. Every physics decision is interpretive.
  - Historical-accuracy claims require citation discipline. Each implemented mechanic should reference the source (interview, paper, footage) it was rebuilt from.
- **Neutral**
  - Documentation-trail research becomes a deliverable — `docs/references/` accumulates source citations per mechanic. Useful for any future writeup about cyrius-bb's design process.

## Alternatives considered

- **Emulator + frame-dump reverse-engineering.** Technically easy. Rejected on both ethical (commercial ROM) and cultural (sovereignty stance includes the build process) grounds.
- **Pick a different brick-breaker (non-Atari) to avoid the IP question.** Considered. Rejected — the point is the specific homage to Breakout as the genre's origin. A generic brick-breaker in Cyrius is not the same statement.
- **Wait for a hypothetical Atari ROM release.** No timeline exists; no indication one will come. Rejected — the homage lands now, while the 50-year anniversary context is active.
- **Commission an Atari license for a legitimate port.** Out of scope for a GPL-3.0-only hobbyist homage; pricing / licensing terms would contradict the free-software distribution model. Rejected.

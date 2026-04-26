# 0002 — Original assets only

**Status**: Accepted
**Date**: 2026-04-24

## Context

[ADR 0001](0001-homage-from-observation.md) committed cyrius-bb to rebuilding mechanics from documented public sources without touching the original ROM. That ADR deliberately scoped *mechanics*, not *assets*. This ADR closes the asset gap.

Breakout's visual and auditory identity — block-pixel sprites, primary-color brick grid, signature paddle-ball sound — is recognizable and specific. Those assets are Atari's (now Atari SA's) property, not ours to ship. The rebuild needs an asset strategy.

## Decision

**Original assets, era-spirit, attribution-clean.** Every shipped asset — sprites, background, UI, sound effects, music — is newly created for this project, commissioned with clear license, or drawn from public-domain reference material. No asset is extracted from the Atari ROM, recreated pixel-identical to the original, or produced by ML-training on Atari-era asset libraries.

**Visual direction:**
- Era-spirit: 1970s arcade aesthetic — bold primaries, chunky pixels, monospace fonts, scanline-adjacent visual effects if they read as era-appropriate.
- Not era-identical: no pixel-matching specific Atari brick layouts, no recreating the exact Breakout color sequence (red/orange/yellow/green/blue rows), no mimicking the cabinet's CRT-specific bloom.
- Original illustration OK: if an artist wants to paint something in the spirit of early-arcade art direction, that's welcome. Contract in `docs/design/commissions/` if applicable.

**Audio direction:**
- New square-wave / simple-FM sound effects synthesized via shravan. No samples, no re-recordings of Atari hardware.
- Music is new original composition (or absent / silent mode, same pattern as cyrius-braid's slot-loaded soundtrack).
- If a user wants to slot-load their own Breakout soundtrack recordings, that's their local choice — we don't ship those files.

**Level data:**
- New level layouts. The first level can be a homage to Breakout's signature layout (8 rows × 14 columns of solid bricks) without being byte-identical.
- Later levels are original design — diagonals, gaps, moving bricks, colored-scoring variants, etc.

Scope of this decision:

- **In**: original art, commissioned work with clear license, public-domain reference material, procedurally-generated art, new original level layouts in the spirit of the genre.
- **Out**: ROM-ripped sprites / audio / palettes; pixel-accurate recreations; ML-generated art trained on Atari-era libraries; level layouts dumped from emulator memory.

## Consequences

- **Positive**
  - Asset posture stays clean. Every shipped file is defensible on its own.
  - Opens the door for community art contributions under the same standard.
  - Commissioning original artists — if budget permits — supports working artists, which is a better homage to the Atari-era designers than copying would be.
- **Negative**
  - Asset production is slower than ROM-ripping would be.
  - Some of Breakout's recognizable "feel" comes from specific pixel layouts + the CRT-cabinet chrome — an original-only approach won't reproduce that 1:1.
- **Neutral**
  - `docs/design/` becomes a real directory with palette references, era-reference links (public-domain museum arcade photos, etc.), and commission scope docs when applicable.

## Alternatives considered

- **Use placeholder assets forever.** Abandons the homage posture. Rejected.
- **ROM-rip and justify as "educational."** Not our call to make; Atari SA holds the copyright. Rejected on ethical grounds.
- **Commission a single artist to do all assets in an Atari-era aesthetic.** Viable if budget appears. Welcome but not required.
- **Procedurally generate everything (no artist at all).** Works for blocks and particles; music and UI ornamentation read as programmer-art without a human eye. Hybrid (procedural + occasional commissioned illustration) is the likely shipping state.
- **ML-generate in the style of Atari art.** Rejected — contradicts the sovereignty stance (ML-derivation pipeline on someone else's work) and would likely implicate Atari SA's moral rights even if statutory copyright is unclear.

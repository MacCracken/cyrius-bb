# 002 — Wiring sankoch + sigil ~4x's the binary (DCE can't trim them)

**Status**: in force as of M5 (v0.6.0).

## The jump

The DCE release binary grew from **117,184 B** (0.5.0) to **454,352 B**
(0.6.0) — ~3.9x — the moment M5 wired in sankoch (compression) + sigil
(integrity hash) for the high-score save file. Every milestone before M5
was bare-stdlib and tracked in the ~100 KB range; M5 is the first that
consumes the shared crates, per [ADR 0003](../adr/0003-self-rolled-primitives.md)
("use them, don't self-roll zip/crypto").

## Why DCE doesn't help

This is the non-obvious part: **`CYRIUS_DCE=1` produces the same size as a
non-DCE build** once these bundles are included. Measured during M5:

- Removing the `compress()` call (keep sigil) — still ~454 KB.
- Removing the `hmac_sha256()` call (keep sankoch) — still ~454 KB.
- `compress(FORMAT_ZLIB, …)` vs `FORMAT_LZ4` vs direct `zlib_compress` —
  all within ~200 B of each other.

The cost is driven by **inclusion**, not by which functions we call. The
sankoch/sigil dist bundles cross-reference broadly (sankoch's `compress`
dispatches to *every* codec backend; sigil wires hash/signature algorithms
through tables/init), so DCE's reachability analysis can't prove the bulk
dead. `_keccak_*` and `deflate_decompress` *do* get marked dead, but enough
of each bundle stays live that the binary lands at ~454 KB regardless.

## Consequence

- The "tiny binary" narrative ends at M5, by design — the size now
  reflects real production compression + crypto, which is the point of the
  stack demo (cyrius-bb proves the Cyrius game stack carries first-party
  crates doing real work).
- Trimming would mean leaner consumption of the bundles (single-codec /
  single-hash entry points that don't drag the dispatch tables) — an
  upstream sankoch/sigil DCE-friendliness concern, not a cyrius-bb one.
  Out of scope for shipping M5; revisit only if binary size becomes a
  release blocker.
- `freelist` / `bigint` / `ct` / `keccak` are in the `[deps].stdlib` list
  only to resolve sigil's signature-path references so the **non-DCE** dev
  build stays warning-free; they don't change the DCE size (that code is
  dead either way).

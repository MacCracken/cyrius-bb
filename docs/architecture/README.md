# Architecture notes

Non-obvious constraints / invariants. Numbered chronologically — never renumber.

Not decisions (those live in [`../adr/`](../adr/)) and not guides. Items here describe *how the world is*.

## Items

- [001 — no-FFI audio](001-no-ffi-audio.md): why audio is self-rolled square-wave PCM over OSS `/dev/dsp`, why there's no music, and the carried-forward console-playback caveat.
- [002 — save-deps binary size](002-save-deps-binary-size.md): why wiring sankoch + sigil at M5 ~4x'd the binary, and why DCE can't trim it.

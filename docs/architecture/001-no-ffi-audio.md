# 001 — Audio is self-rolled square-wave PCM (no FFI, no OGG decoder)

**Status**: in force as of M4 (v0.5.0). **Updated 2026-06-29**: the
*synthesis* half stays self-rolled, but the *playback* sink moved from
OSS `/dev/dsp` to **vani-core** (vendored `vendor/vani-core.cyr`) — direct
ALSA PCM ioctls in pure Cyrius, no FFI. See "How the world actually is".

## The constraint

cyrius-bb has two hard rules that together box in how audio works:

- **No FFI** (CLAUDE.md, AGNOS-wide). PulseAudio / PipeWire (C libraries)
  stay off the table. ALSA itself is *not* a C library — it is a kernel
  ioctl interface — and **vani** speaks it directly in pure Cyrius, so a
  vani-backed ALSA sink is no-FFI-clean. (vani was not consumable when this
  ADR was first written; the synthesis rationale below is unaffected.)
- **shravan is not consumable.** The intended audio crate is pinned at
  v4.10.3 with no `dist/shravan.cyr` bundle (see `cyrius.cyml` pending-deps);
  it needs an upstream toolchain bump + bundle before it can be wired.

So, exactly as the renderer was self-rolled rather than waiting on mabda
([ADR 0003](../adr/0003-self-rolled-primitives.md)), audio is self-rolled
on bare stdlib.

## How the world actually is

- **Synthesis** (`src/audio.cyr`): square-wave SFX with linear frequency +
  amplitude sweeps. Unsigned 8-bit mono PCM at `AUDIO_RATE` (11025 Hz) —
  the classic OSS default, so one buffer is byte-identical to `/dev/dsp`
  data *and* to 8-bit WAV data. This is the tested core (sample-assertable,
  no device I/O), mirroring `framebuf.cyr`. Square beeps are also faithful
  to Breakout's 1976 sound, so this is era-spirit, not a compromise.
- **Playback** (`src/sound.cyr`): best-effort **ALSA** via vani-core's
  `audio_*` shim (`audio_open_playback` / `set_params` / `prepare` /
  `write_bytes` / `drain` / `close`) → `/dev/snd/pcmC{card}D{device}p`. It
  is the `present.cyr` analogue — environment-specific, **not unit-tested**,
  and a no-op (silent game) when no device opens (`audio_open_playback`
  returns 0). Replaces the prior OSS `/dev/dsp` sink, which was silent on
  modern ALSA-only systems. Card/device default to 1/0 (vani's verified
  analog target); edit the `SoundDev` constants in `sound.cyr` per box.
- **Verification** (`audio_write_wav` + `programs/audio_demo.cyr`): dumps
  each SFX to a playable WAV, so the sounds can actually be *heard*
  (`ffplay build/sfx_*.wav`) without a device — the ear-equivalent of the
  renderer's eyeball-able PPM frame dumps.

## Consequences / gotchas

- **Now audible on modern desktops.** The vani ALSA sink plays on any box
  with a sound card — no OSS shim / `padsp` needed (the prior `/dev/dsp`
  path was silent on modern ALSA-only systems). In-game audio is still
  device-gated (no card ⇒ silent no-op, like `/dev/fb0` present); the WAV
  dump remains the always-works fallback for confirming synthesis offline.
- **No mixer yet.** Playback is fire-and-forget; concurrent SFX serialise
  rather than mix. A real voice mixer is a playtest-driven refinement, not
  an M4 requirement.
- **No music.** Slot-loaded `.ogg` music (roadmap M4) needs an OGG Vorbis
  decoder, which is infeasible without FFI / a decoder crate. Deferred; if
  revisited, the format will be **WAV** (header-parseable with no decoder),
  not OGG. The SFX-focused M4 acceptance ("audio adds to the feel") does
  not depend on music.
- **Raw-hardware rate.** vani opens the PCM device directly (no plug /
  resample layer), so the 11025 Hz mono synth rate must be one the card
  accepts. Most HDA codecs negotiate it; if a device rejects it,
  `audio_set_params` fails and the game falls back to silent — bump the
  synth rate or gate on `audio_can_set_params` as a playtest fix.

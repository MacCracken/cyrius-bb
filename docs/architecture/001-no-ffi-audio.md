# 001 — Audio is self-rolled square-wave PCM (no FFI, no OGG decoder)

**Status**: in force as of M4 (v0.5.0).

## The constraint

cyrius-bb has two hard rules that together box in how audio works:

- **No FFI** (CLAUDE.md, AGNOS-wide). So ALSA / PulseAudio / PipeWire —
  all C libraries — are off the table.
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
- **Playback** (`src/sound.cyr`): best-effort **OSS `/dev/dsp`**, opened
  write-only + non-blocking. This is the only no-FFI real-time sink. It is
  the `present.cyr` analogue — environment-specific, **not unit-tested**,
  and a no-op (silent game) if the device will not open.
- **Verification** (`audio_write_wav` + `programs/audio_demo.cyr`): dumps
  each SFX to a playable WAV, so the sounds can actually be *heard*
  (`ffplay build/sfx_*.wav`) without a device — the ear-equivalent of the
  renderer's eyeball-able PPM frame dumps.

## Consequences / gotchas

- **Modern desktops have no `/dev/dsp`.** Real-time audio needs an OSS
  shim (`padsp ./cyrius-bb`) or a console/kernel with OSS. Until verified
  on such a setup, in-game audio is build-verified only — same carried-
  forward status as `/dev/fb0` present. The WAV dump is the always-works
  fallback for confirming the synthesis is correct.
- **No mixer yet.** Playback is fire-and-forget; concurrent SFX serialise
  rather than mix. A real voice mixer is a playtest-driven refinement, not
  an M4 requirement.
- **No music.** Slot-loaded `.ogg` music (roadmap M4) needs an OGG Vorbis
  decoder, which is infeasible without FFI / a decoder crate. Deferred; if
  revisited, the format will be **WAV** (header-parseable with no decoder),
  not OGG. The SFX-focused M4 acceptance ("audio adds to the feel") does
  not depend on music.
- The OSS ioctl numbers (`SNDCTL_DSP_SPEED = 0xC0045002`) are hardcoded and
  best-effort; if the device's default rate differs and the ioctl is
  ignored, pitch will be off — a console-playtest fix.

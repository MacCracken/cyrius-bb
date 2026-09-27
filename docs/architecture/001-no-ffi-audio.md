# 001 — Audio is self-rolled square-wave PCM (no FFI, no OGG decoder)

**Status**: in force as of M4 (v0.5.0). **Updated 2026-06-29**: the
*synthesis* half stays self-rolled, but the *playback* sink moved from
OSS `/dev/dsp` to **vani-core** (vendored `vendor/vani-core.cyr`) — direct
ALSA PCM ioctls in pure Cyrius, no FFI. **Updated 2026-09-26**: a hardware
probe showed that sink had never played. The synth now renders the device's
own format (48 kHz S16_LE stereo), and the sink recovers the stream between
cues. See "How the world actually is".

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
  amplitude sweeps, pre-rendered once into a heap cache (~255 KB) by
  `audio_init`. The format is the playback device's own: **48 kHz, S16_LE,
  interleaved stereo** (L == R). A raw ALSA hw PCM converts nothing, so the
  synth has to produce what the codec takes. One buffer is byte-identical to
  the PCM the device receives *and* to 16-bit WAV data. This is the tested
  core (sample-assertable, no device I/O), mirroring `framebuf.cyr`. Square
  beeps are also faithful to Breakout's 1976 sound, so this is era-spirit,
  not a compromise.
- **Playback** (`src/sound.cyr`): best-effort **ALSA** via vani-core's
  `audio_*` shim → `/dev/snd/pcmC{card}D{device}p`. `sound_open` programs the
  format with `audio_set_params_fmt(…, SND_PCM_FORMAT_S16_LE, …)` on an
  explicit ring of 32 × 1024 frames, then sets silence-filled sw params.
  `sound_play` writes a cached cue with `audio_write`, and `sound_close`
  drains and closes. It is the `present.cyr` analogue: environment-specific,
  its ioctl path is **not unit-tested**, and it is a no-op (silent game) when
  no device opens or the device refuses the format. The pure pieces (the
  retry policy, the ring shape) are unit-tested. Replaces the prior OSS
  `/dev/dsp` sink, which was silent on modern ALSA-only systems. Card/device
  default to 1/0 (vani's verified analog target); edit the `SoundDev`
  constants in `sound.cyr` per box.
- **Stream model.** The ring is fed only when a cue fires, so every cue ends
  with it running dry. The kernel then stops the stream (XRUN: the default
  `stop_threshold` is the ring size), and the next write fails `-EPIPE`, so
  `sound_play` re-prepares and retries once. The ring (~683 ms) holds the
  worst one-tick burst, so `audio_write`, a blocking ioctl, never waits on an
  idle ring. The sw params keep the ring past the queued audio zeroed, so the
  DMA never replays an earlier cue's leftovers before the underrun is noticed.
- **Verification** (`audio_write_wav` + `programs/audio_demo.cyr`): dumps
  each SFX to a playable WAV, so the sounds can actually be *heard*
  (`ffplay build/sfx_*.wav`) without the game's device path — the
  ear-equivalent of the renderer's eyeball-able PPM frame dumps.

## Consequences / gotchas

- **No sound server, no shim.** The vani ALSA sink needs no OSS shim /
  `padsp` (the prior `/dev/dsp` path was silent on modern ALSA-only
  systems). It does need the card to itself and an unmuted codec mixer (the
  last two gotchas below). In-game audio is still device-gated (no card ⇒
  silent no-op, like `/dev/fb0` present); the WAV dump remains the
  always-works fallback for confirming synthesis offline.
- **No mixer yet.** Playback is fire-and-forget; concurrent SFX queue back
  to back in the ring rather than mix. A real voice mixer is a
  playtest-driven refinement, not an M4 requirement.
- **No music.** Slot-loaded `.ogg` music (roadmap M4) needs an OGG Vorbis
  decoder, which is infeasible without FFI / a decoder crate. Deferred; if
  revisited, the format will be **WAV** (header-parseable with no decoder),
  not OGG. The SFX-focused M4 acceptance ("audio adds to the feel") does
  not depend on music.
- **Raw-hardware format.** vani opens the PCM device directly (no plug /
  resample layer), so the synth's format must be one the codec takes as-is.
  The dev box's ALC897 (card 1) accepts only S16_LE / S32_LE, exactly 2
  channels, at 44.1 kHz and up. It refused the 11025 Hz unsigned 8-bit mono
  this synth rendered through 0.8.3 with `-EINVAL`, which went unchecked, so
  the game was silent from 0.8.0 until the 2026-09-26 fix. 48 kHz S16 stereo
  is the HD-Audio baseline format. A device that refuses it leaves the
  game silent (`sound_open` closes it) rather than writing to an unconfigured
  PCM.
- **The codec's own mixer still applies.** A raw hw PCM bypasses PipeWire's
  volume, not the codec's amps. With no desktop session on the seat, nothing
  may have set them. On the dev box with only the login greeter active, the
  DAC sat at −65 dB and both output pins were muted, which makes a working
  playback path inaudible. Set the mixer before judging the audio (log in on
  the seat, then `wpctl` or `alsamixer`).
- **A busy card stalls start-up.** vani opens the PCM without `O_NONBLOCK`,
  so while another client holds the card (PipeWire playing something, another
  game), `sound_open` waits for it instead of failing. It runs before the
  menu, so the game hangs until the card is free (measured: a 2.7 s wait on a
  card another process held for 3 s). PipeWire's default is to release an
  idle card after 5 s. The fix belongs upstream in vani's
  `audio_open_playback`.

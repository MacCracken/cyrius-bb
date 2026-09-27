# vendor/

Third-party single-file snapshots that are **deliberately committed**
(unlike `lib/`, the regenerable stdlib snapshot, which is gitignored).

## `vani-core.cyr`

- **Source**: [vani](https://github.com/MacCracken/vani) `dist/vani-core.cyr`
  (the `core` profile — the playback-only ALSA PCM shim: the `audio_*` API).
- **Version**: 1.2.5, copied from the vani `1.2.5` tag (vani pins cyrius 6.6.2;
  we build on 6.6.6).
- **Consumed by**: `src/sound.cyr`, via `audio_open_playback`,
  `audio_set_params_fmt` (explicit `SND_PCM_FORMAT_S16_LE`; vani ≥ 1.2.0),
  `audio_prepare`, `audio_write`, `audio_drain` and `audio_close`. It also
  uses `audio_fd` plus the `SNDRV_PCM_IOCTL_SW_PARAMS` / `AlsaSwParamsLayout`
  constants for the silence-filled sw params (`sound_set_sw_silence`), because
  vani's `audio_set_sw_params` pins the silence fields to 0. A refresh must
  keep those names. `include "vendor/vani-core.cyr"` sits before
  `src/sound.cyr` in `src/main.cyr`'s chain and in `tests/cyrius-bb.tcyr`. It
  needs only the `syscalls` + `string` + `alloc` stdlib modules (vani's
  `dist/vani-core.deps` sidecar), all already in `cyrius.cyml [deps].stdlib`.
- **Why vendored instead of a `[deps.vani]` git dependency**: keeps the
  "bare stdlib / zero external deps" build lean — resolving vani as a git dep
  pulls its whole manifest tree (yukti, patra, sakshi) into `lib/`. The core
  profile is self-contained (raw ALSA over syscalls), so committing the one
  file is the lean, reproducible choice. Same pattern cyrius-polyomino uses.
- **Replaces**: the legacy OSS `/dev/dsp` sink, which was silent on modern
  ALSA-only systems. See `docs/architecture/001-no-ffi-audio.md`.

### Refreshing to a newer vani

```sh
# read the file from the release tag, not the working tree (which may be dirty):
git -C /path/to/vani show <tag>:dist/vani-core.cyr > vendor/vani-core.cyr
# bump the Version line above; no call-site changes if the audio_* API is stable
```

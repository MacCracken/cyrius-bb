# vendor/

Third-party single-file snapshots that are **deliberately committed**
(unlike `lib/`, the regenerable stdlib snapshot, which is gitignored).

## `vani-core.cyr`

- **Source**: [vani](https://github.com/MacCracken/vani) `dist/vani-core.cyr`
  (the `core` profile — the playback-only ALSA PCM shim: the `audio_*` API).
- **Version**: 1.2.5, copied from the vani `1.2.5` tag (vani pins cyrius 6.6.2;
  we build on 6.6.6).
- **Consumed by**: `src/sound.cyr` (via `audio_open_playback` /
  `audio_set_params` / `audio_prepare` / `audio_write_bytes` / `audio_drain` /
  `audio_close`). `include "vendor/vani-core.cyr"` sits before `src/sound.cyr`
  in `src/main.cyr`'s chain; it needs only the `syscalls` + `string` + `alloc`
  stdlib modules (vani's `dist/vani-core.deps` sidecar), all already in
  `cyrius.cyml [deps].stdlib`.
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

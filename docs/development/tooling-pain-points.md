# Cyrius Tooling Pain Points (Field Notes)

Captured 2026-04-26 while wiring `cyrius.cyml` deps for cyrius-bb.
Toolchain: cyrius 5.7.7. Internal-tool dogfood targets: `cyrius`,
`owl`, `cyim`. This is a running log — append, don't rewrite.

## cyrius deps

### P1 — resolver doesn't validate `modules` paths exist

**Repro.** Add a `[deps.hisab]` block pinned to tag `2.2.0` with
`modules = ["dist/hisab.cyr"]`. Hisab's 2.2.0 tag has no `dist/`
directory — only `src/`, `lib/`, `examples/`.

**Observed.** `cyrius deps` reports `5 deps resolved`, exit 0.
`lib/hisab.cyr` is created as a symlink pointing to
`~/.cyrius/deps/hisab/2.2.0/dist/hisab.cyr` — a path that doesn't
exist. `cyrius build` against a `main.cyr` that doesn't include
hisab still passes because the dangling link is never dereferenced.

**Expected.** Resolver should `stat` each module path inside the
clone and fail loudly: `error: [deps.hisab] modules entry
"dist/hisab.cyr" not found at tag 2.2.0`. A successful "N deps
resolved" line should never coexist with a dangling `lib/` symlink.

**Severity.** High. The bug is silent until something actually
includes the dep, and the build error at that point doesn't blame
the resolver — it blames the missing include. Multi-hour debug
trap.

### P2 — `cyrius deps --help` runs the resolver instead of printing help

**Repro.** `cyrius deps --help`.

**Observed.** Treats `--help` as a no-op flag and runs resolution
(`4 deps resolved`).

**Expected.** Print subcommand-specific help: what flags exist,
where the cache lives, how to force re-clone, how to clear stale
entries. Same shape as `git push --help` or `cargo build --help`.

### P3 — `cyrius deps` not listed in top-level help

**Repro.** `cyrius help` (or `cyrius --help`).

**Observed.** Sections: Build, Project, Quality, Testing,
Interactive, Tools, Info. No Deps section. `deps` isn't mentioned
anywhere in the top-level usage page.

**Expected.** Either a Deps section, or fold under Project alongside
`init`. Discoverability matters more than where it lives.

### P4 — first-resolve count off-by-one

**Repro.** Fresh project, declare 4 `[deps.NAME]` blocks. Run
`cyrius deps` twice.

**Observed.** First run: `5 deps resolved`. Second run (cache
warm): `4 deps resolved`. The count differs by 1 between cold and
warm cache.

**Expected.** Same number both runs. Likely the cold path is
counting either the stdlib batch or one of the dep clone+symlink
steps as an extra unit; the count should report distinct deps, not
operations.

### P5 — lockfile flag exists but is opt-in and undocumented

**Repro.** Run `cyrius deps` with `[deps.NAME]` blocks declared.

**Observed (default).** No `cyrius.lock` produced. Bare `cyrius
deps` only prints `N deps resolved` and writes `lib/` symlinks.

**Observed (`--lock`).** `cyrius deps --lock` does generate
`cyrius.lock` — sha256 + `  lib/<name>.cyr` per line, byte-format
matches yukti's. Output: `cyrius.lock: N deps locked`.

**Discoverability.** `--lock` is not in `cyrius help`, not in
`cyrius deps --help` (since that just runs the resolver — see P2),
not in any usage error. Found only by `strings $(which cyrius) |
grep lock`. Sibling flags surfaced the same way: `--update`,
`--frozen` (both accepted, behavior not documented).

**Expected.** `cyrius deps` should write/refresh `cyrius.lock` on
every successful resolve **by default** — that's the user's stated
mental model and the pattern other dep tools follow (cargo, npm,
go mod). `--no-lock` for the rare opt-out. At minimum, the
existing `--lock` / `--update` / `--frozen` triad should appear in
help. The lockfile is the only thing making "pinned deps" actually
reproducible across machines — making it opt-in defeats the
purpose for new projects whose authors don't know the flag exists.

## mabda

### P6 — soorat (windowing) is named in mabda but absent from project docs

**Repro.** `grep -nE "soorat" lib/mabda.cyr`. Mabda comments explicitly
say *"Consumers like soorat provide the handle"* (mabda is headless;
it wraps wgpu but does not open a window).

**Observed.** soorat exists at `~/Repos/soorat/` but is still Rust
(`Cargo.toml` + `target/` + `rust-toolchain.toml`). It is **not**
mentioned in `cyrius-bb/CLAUDE.md`, in `docs/development/roadmap.md`,
or in `docs/development/state.md`. The project's stated dep chain
lists mabda → kiran → impetus → shravan → hisab → sankoch → sigil;
soorat is invisible.

**Impact.** M1 acceptance (`Render via mabda`) is *not* unblocked just
because mabda exists in Cyrius. The full render path needs soorat
(window + surface handle) → mabda (wgpu shim) → kiran (engine /
input / event loop). Without soorat in Cyrius, even a minimal
"open a coloured window" smoke test cannot be done in pure Cyrius.

**Expected.** CLAUDE.md's dep-chain comment should list soorat
alongside kiran/impetus as Rust-pending. roadmap.md M1 should call
out the soorat port as a prerequisite for the render acceptance
criterion. state.md should track the port status of the three
Rust-pending deps.

### P7 (minor) — `gpu_err_name` returns a description, not the symbolic name

**Repro.** `gpu_err_name(GPU_ERR_NONE)` → `"none"` (not `"GPU_ERR_NONE"`).

**Observed.** The function name reads as "give me the name of this
error" but returns a human-readable description. Reasonable choice,
but the name is ambiguous — readers expect either the symbolic
constant name or a description, and `_name` reads as the former.

**Expected.** Rename to `gpu_err_describe` or `gpu_err_message` to
match what's actually returned, and (separately) add a
`gpu_err_symbol` that returns the constant string if needed. Tiny
papercut, not a blocker — but it's the kind of API friction that
adds up across a 432-fn surface.

## Wins

### dep-include path works end-to-end for pure-Cyrius mabda calls

**Setup.** `[deps.mabda]` git/tag/modules block in cyrius.cyml,
`cyrius deps --lock` to resolve + lock, single mabda call in
`src/main.cyr` (`gpu_err_name(GPU_ERR_NONE)`), `cyrius build`,
run.

**Result.** Build green (only sigil dead-code warnings — those are
unused-API noise, not errors), runs clean, prints expected output.
The whole pipeline — declare dep → resolve → symlink into `lib/`
→ auto-include → compile → link → run — Just Works for pure-Cyrius
mabda functions. The friction we found (P1-P7) is real but does
not block this path.

### `cyim --write` heredoc handles em-dashes and unicode cleanly

The `main.cyr` body included an em-dash (`—`) inside a string
literal. `cyim --write <file>` from a heredoc round-tripped it
byte-for-byte. No escaping required.

## owl

### Note — `--line-range=A:B` is the head/tail substitute

`owl` has no `-n` (head) or `-N` (tail) flag the way `head(1)` and
`tail(1)` do — `--line-range=:N` is "first N lines" and
`--line-range=A:` is "from line A to end". Works fine once you
know it; the help text could call out the head/tail pattern as the
intended idiom (`# head equivalent: owl --line-range=:20 file`).

### Note — `-p` strips decorations cleanly

`-p` (plain) is the right output mode when piping owl's output into
something else or capturing it. `--paging=never` was unnecessary
in non-interactive contexts; owl auto-detects.

## cyim

### Win — `--expect` / `--expect-1` / `--expect-N=<n>` post-save assertions

These are *better* than the built-in Edit tool's "old_string must
be unique" rule. Contract is explicit and post-verifiable, exit 6
on miss. Used `--expect='\[deps\.hisab\]'` and
`--expect-not='"sigil",'` on the cyrius.cyml rewrite — both fired
correctly.

### Note — no multi-edit-in-one-call mode

For "do these N distinct substitutions on one file" the workflow
is N sequential `cyim --replace` invocations. Worked fine here
(three chained edits with `&&` to move the hisab block) but a
batch mode (read a list of `<old>=><new>` pairs) would be cleaner
for larger refactors.

### Note — `--replace` requires uniqueness, surfaces collisions early

`cyim --replace` errored cleanly when the `<old>` string would have
matched zero or many places. That's the right default. Combined
with `--expect-1` it gives belt-and-suspenders.

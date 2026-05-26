# Cyrius Tooling Pain Points (Field Notes)

Captured 2026-04-26 while wiring `cyrius.cyml` deps for cyrius-bb.
Dogfood targets: `cyrius`, `owl`, `cyim`. This is a running log —
append, don't rewrite.

**Toolchain at last sweep:** cyrius 5.7.11 + cyim 1.1.3 + owl 1.1.6.
**Pin now at cyrius 6.0.1** (bumped 2026-05-25) — the open `cyrius deps` items below have **not** yet been re-swept against 6.0.1; carried forward pending a real re-run.

## Status

| ID  | Area         | Title                                                | Severity | Status |
|-----|--------------|------------------------------------------------------|----------|--------|
| P1  | cyrius deps  | resolver doesn't validate `modules` paths            | HIGH     | ❌ open |
| P2  | cyrius deps  | `cyrius deps --help` runs the resolver               | MED-docs | ❌ open |
| P3  | cyrius deps  | `cyrius deps` not in top-level help                  | MED-docs | ❌ open |
| P4  | cyrius deps  | first-resolve count off-by-one                       | LOW      | ✅ fixed in cyrius 5.7.8 |
| P5  | cyrius deps  | lockfile not auto-generated                          | MED      | ❌ open — paired with P8 |
| P6  | mabda / docs | soorat absent from cyrius-bb dep-chain docs          | project  | ❌ open (project-side) |
| P7  | mabda        | `gpu_err_name` returns description, not symbolic name | LOW     | ❌ open |
| P8  | cyrius deps  | `cyrius deps --lock` regressed from cold             | HIGH     | 🆕 new since cyrius 5.7.8 — paired with P5 |
| —   | owl          | head/tail idiom undocumented                         | LOW      | ✅ fixed in owl 1.1.6 |
| —   | cyim         | no multi-edit-in-one-call mode                       | LOW      | ✅ fixed in cyim 1.1.2 |
| —   | cyim         | multiple `--expect=` flags — semantics unclear       | LOW      | ✅ fixed in cyim 1.1.3 (rejected with `duplicate flag`) |
| —   | cyim         | `--grep` is literal substring; needs `--find` regex + `--regex=<flavor>` | LOW      | ❌ open (design agreed 2026-04-26) |

**Severity legend.** HIGH = silent corruption or multi-hour debug
trap. MED = workflow blocker with workaround, or discoverability
gap that costs time. LOW = naming / cosmetic / minor friction.
MED-docs = help-system fix only; behaviour is fine.

## For the cyrius lang agent

Suggested order of attack when a work window opens:

1. **P1** — single-file fix in the deps resolver (`stat` each
   module path inside the clone before symlinking; fail loudly on
   miss). High impact, isolated change. Ship first.
2. **P5 + P8 together** — these are clearly the same in-flight
   feature (lockfile-by-default). The current half-state — `--lock`
   no longer resolves cold *and* plain `cyrius deps` still doesn't
   write the lock — is worse than either endpoint. Pick one shape
   and finish it. P5's "Expected" lays out the cargo/npm/go-mod
   convention.
3. **P2 + P3** — one help-system pass covers both: surface
   `cyrius deps` in the top-level help block, and have
   `cyrius deps --help` print subcommand help (flags, cache
   location, sibling flags `--update` / `--frozen` / `--lock`).
4. **P7** — 1-line API rename (`gpu_err_name` → `gpu_err_describe`),
   only worth doing if the mabda v3.0 backend swap window is
   already open and breaking changes are landing anyway.
5. **P6** — project-side action (CLAUDE.md / roadmap.md / state.md
   updates in cyrius-bb). Not a toolchain fix.

The `cyim` "multiple `--expect=`" item needs a focused test
before it becomes a real pain point — flagged so it isn't lost.

## cyrius deps

### P1 — resolver doesn't validate `modules` paths exist  (HIGH)

**Repro.** Add a `[deps.hisab]` block pinned to tag `2.2.0` with
`modules = ["dist/hisab.cyr"]`. Hisab's 2.2.0 tag has no `dist/`
directory — only `src/`, `lib/`, `examples/`.

**Observed.** `cyrius deps` reports `4 deps resolved`, exit 0.
`lib/hisab.cyr` is created as a symlink pointing to
`~/.cyrius/deps/hisab/2.2.0/dist/hisab.cyr` — a path that doesn't
exist. `cyrius build` against a `main.cyr` that doesn't include
hisab still passes because the dangling link is never dereferenced.

**Expected.** Resolver should `stat` each module path inside the
clone and fail loudly: `error: [deps.hisab] modules entry
"dist/hisab.cyr" not found at tag 2.2.0`. A successful "N deps
resolved" line should never coexist with a dangling `lib/` symlink.

**Why HIGH.** The bug is silent until something actually includes
the dep, and the build error at that point doesn't blame the
resolver — it blames the missing include. Multi-hour debug trap.
Re-confirmed against cyrius 5.7.8.

### P2 — `cyrius deps --help` runs the resolver instead of printing help  (MED-docs)

**Repro.** `cyrius deps --help`.

**Observed.** Treats `--help` as a no-op flag and runs resolution
(`3 deps resolved`).

**Expected.** Print subcommand-specific help: what flags exist
(`--lock`, `--update`, `--frozen` discovered via `strings`), where
the cache lives (`~/.cyrius/deps/<name>/<tag>/`), how to force
re-clone, how to clear stale entries. Same shape as `git push --help`
or `cargo build --help`.

### P3 — `cyrius deps` not listed in top-level help  (MED-docs)

**Repro.** `cyrius help` (or `cyrius --help`).

**Observed.** Sections: Build, Project, Quality, Testing,
Interactive, Tools, Info. No Deps section. `deps` isn't mentioned
anywhere in the top-level usage page.

**Expected.** Either a Deps section, or fold under Project alongside
`init`. Discoverability matters more than where it lives.

### P4 — first-resolve count off-by-one  ✅ RESOLVED in cyrius 5.7.8

**Repro (5.7.7).** Fresh project, declare 4 `[deps.NAME]` blocks.
Run `cyrius deps` twice.

**Observed (5.7.7).** First run: `5 deps resolved`. Second run
(cache warm): `4 deps resolved`. Count differed by 1 between cold
and warm cache.

**Resolution.** Re-run against 5.7.8: cold and warm both report
`3 deps resolved` (project has 3 live `[deps.NAME]` blocks).
Count is stable across runs.

### P5 — lockfile flag exists but is opt-in and undocumented  (MED — paired with P8)

**Repro.** Run `cyrius deps` with `[deps.NAME]` blocks declared.

**Observed (default).** No `cyrius.lock` produced. Bare `cyrius
deps` only prints `N deps resolved` and writes `lib/` symlinks.

**Observed (`--lock` in 5.7.7).** `cyrius deps --lock` generated
`cyrius.lock` correctly — sha256 + `  lib/<name>.cyr` per line,
byte-format matches yukti's. Output: `cyrius.lock: N deps locked`.

**Observed (`--lock` in 5.7.8).** Regressed — see P8.

**Discoverability.** `--lock` is not in `cyrius help`, not in
`cyrius deps --help` (since that just runs the resolver — see P2),
not in any usage error. Found only by `strings $(which cyrius) |
grep lock`. Sibling flags surfaced the same way: `--update`,
`--frozen` (both accepted, behavior not documented).

**Expected.** `cyrius deps` should write/refresh `cyrius.lock` on
every successful resolve **by default** — that's the user's stated
mental model and the pattern other dep tools follow (cargo, npm,
go mod). `--no-lock` for the rare opt-out. The lockfile is the
only thing making "pinned deps" actually reproducible across
machines — making it opt-in defeats the purpose for new projects
whose authors don't know the flag exists.

### P8 — `cyrius deps --lock` no longer resolves from cold  🆕 NEW in cyrius 5.7.8 (HIGH — paired with P5)

**Repro.** Clean slate (`rm cyrius.lock lib/<dep>.cyr` for each
git dep). Run `cyrius deps --lock`.

**Observed (5.7.7).** Single command resolved and locked: output
`cyrius.lock: 3 deps locked`, lockfile written with sha256s.

**Observed (5.7.8).** Same command from cold reports `cyrius.lock:
0 deps locked` and writes an **empty** lockfile. The `lib/`
symlinks *are* created as a side effect, but the lock step counts
zero.

**Workaround.** Two-step: `cyrius deps && cyrius deps --lock`. The
first run resolves into `lib/`; the second locks what now exists.
Produces the correct 3-line lockfile.

**Likely cause.** `--lock` semantics changed from "resolve+lock"
to "lock-already-resolved". Probably intentional groundwork for
making `cyrius deps` lock-by-default (the eventual P5 fix), but
the transition leaves `--lock` doing the wrong thing on cold runs.

**Why HIGH.** An empty lockfile written silently looks like
success. CI that runs `cyrius deps --lock` and commits the result
would commit an empty file — the integrity guarantee disappears
without any visible error.

**Expected.** Either (a) make `--lock` resolve when needed, same
as 5.7.7; or (b) ship the P5 fix at the same time so plain
`cyrius deps` writes the lockfile and `--lock` becomes redundant.
Halfway between is worse than either end.

## mabda

### P6 — soorat (windowing) is named in mabda but absent from project docs  (project-side)

**Repro.** `grep -nE "soorat" lib/mabda.cyr`. Mabda comments
explicitly say *"Consumers like soorat provide the handle"*
(mabda is headless; it wraps wgpu but does not open a window).

**Observed.** soorat exists at `~/Repos/soorat/` but is still
Rust (`Cargo.toml` + `target/` + `rust-toolchain.toml`). It is
**not** mentioned in `cyrius-bb/CLAUDE.md`, in
`docs/development/roadmap.md`, or in
`docs/development/state.md`. The project's stated dep chain lists
mabda → kiran → impetus → shravan → hisab → sankoch → sigil;
soorat is invisible.

**Impact.** M1 acceptance (`Render via mabda`) is *not* unblocked
just because mabda exists in Cyrius. The full render path needs
soorat (window + surface handle) → mabda (wgpu shim) → kiran
(engine / input / event loop). Without soorat in Cyrius, even a
minimal "open a coloured window" smoke test cannot be done in
pure Cyrius.

**Expected.** CLAUDE.md's dep-chain comment should list soorat
alongside kiran/impetus as Rust-pending. roadmap.md M1 should call
out the soorat port as a prerequisite for the render acceptance
criterion. state.md should track the port status of the three
Rust-pending deps.

### P7 (minor) — `gpu_err_name` returns a description, not the symbolic name  (LOW)

**Repro.** `gpu_err_name(GPU_ERR_NONE)` → `"none"` (not
`"GPU_ERR_NONE"`).

**Observed.** The function name reads as "give me the name of this
error" but returns a human-readable description. Reasonable choice,
but the name is ambiguous — readers expect either the symbolic
constant name or a description, and `_name` reads as the former.

**Expected.** Rename to `gpu_err_describe` or `gpu_err_message`
to match what's actually returned, and (separately) add a
`gpu_err_symbol` that returns the constant string if needed.
Tiny papercut, not a blocker — but it's the kind of API friction
that adds up across a 432-fn surface.

**User response (2026-04-26).** Deferred. mabda 3.0 is focused on
pure-Cyrius GPU-native (no wgpu shim); breaking renames there
would add noise to a bigger pivot. Reconsider for mabda 3.0.1.

## owl

### Note — `--line-range=A:B` is the head/tail substitute  ✅ RESOLVED in owl 1.1.6

`owl` has no `-n` (head) or `-N` (tail) flag the way `head(1)` and
`tail(1)` do — `--line-range=:N` is "first N lines" and
`--line-range=A:` is "from line A to end".

**Resolution.** owl 1.1.6 added `head -n N idiom: --line-range=:N`
inline in the `--line-range` help line. Discoverability fixed
without expanding the flag surface — good shape.

### Note — `-p` strips decorations cleanly

`-p` (plain) is the right output mode when piping owl's output
into something else or capturing it. `--paging=never` was
unnecessary in non-interactive contexts; owl auto-detects.

## cyim

### Win — `--expect` / `--expect-1` / `--expect-N=<n>` post-save assertions

These are *better* than the built-in Edit tool's "old_string must
be unique" rule. Contract is explicit and post-verifiable, exit 6
on miss. Used `--expect='\[deps\.hisab\]'` and
`--expect-not='"sigil",'` on the cyrius.cyml rewrite — both fired
correctly.

### Note — multi-edit-in-one-call mode  ✅ RESOLVED in cyim 1.1.2

**Resolution.** cyim 1.1.2 added `--batch <file>`: reads NUL-
separated OLD/NEW pairs from stdin, applies them atomically.
With `--all` each pair runs as `--replace-all`; without it each
pair is a first-match `--replace`. `--wc` / `--expect=` /
`--expect-not=` all apply post-batch. Smoke-tested with two pairs
on a 3-line file: fox→hawk (×2) and dog→wolf (×1) landed in one
transaction.

**Bonus property.** `--batch` aborts atomically if any OLD pair
doesn't match — caught a stray-backtick typo in one of the OLD
strings without partially editing the file. Right behaviour.

This section was itself updated using `cyim --batch` — the new
feature marked its own resolution.

### Note — `--replace` requires uniqueness, surfaces collisions early

`cyim --replace` errored cleanly when the `<old>` string would
have matched zero or many places. That's the right default.
Combined with `--expect-1` it gives belt-and-suspenders.

### Note — `--grep` does substring match, not regex  (LOW, refined 2026-04-26 against cyim 1.1.3)

**Repro.** Create a file with two relevant lines:
  - `^X is the first line literally` (literal caret)
  - `X starts the third line without caret` (no caret)

Run `cyim --grep '^X' file`.

**Observed.** Matches line 1 only — the line that contains the
literal characters `^X` as a substring. Line 3 (which would match
if `^` were a start-of-line regex anchor) is not returned. Same
behavior in cyim 1.1.2 and 1.1.3.

**Conclusion.** `--grep` is doing literal substring matching, not
regex. The flag name `--grep` and the placeholder `<pattern>` in
`--help` both imply regex semantics, which is the misleading bit.

**Expected (design agreed with user, 2026-04-26).** Two new flags:

- `cyim --find <pattern> <file>` — regex search (replaces or
  supplements `--grep`). Anchors `^` and `$` honored, standard
  metacharacter set documented in `--help`.
- `cyim --regex=<flavor>` — optional flavor selector. Pick a
  default (likely POSIX ERE or a small PCRE subset) and let
  callers override per invocation when they need a different
  flavor.

Keeps `--grep` available for literal substring callers if
back-compat matters; otherwise `--grep` can be retired in favor
of `--find` once consumers migrate. The naming split (`--find`
for the verb, `--regex` for the flavor knob) avoids overloading
one flag with two responsibilities.

**Side-effect noticed.** This file already has 3 occurrences of
the literal string `^## ` (inside code spans documenting this
very issue), so `cyim --grep '^## ' docs/development/tooling-pain-points.md`
now returns 3 hits — but those are the documentation, not the
actual section headers. Self-referential confusion is a small
tell that `--grep` semantics differ from user expectation.

### Note — multiple `--expect=` flags on one invocation  ✅ RESOLVED in cyim 1.1.3

**Original observation.** During the cyrius 5.7.8 sweep three
`--expect=<pat>` flags were passed to a single `cyim --batch` call.
Behaviour was unverifiable — silent shrug, no clear semantics.
Originally requested either (a) AND across patterns, or (b)
explicit rejection of multiple instances.

**Resolution.** cyim 1.1.3 chose (b) and ships a clear error:
`cyim: duplicate flag: --expect`, exit 2, file untouched. Atomic
rejection, no half-edit. Cleaner than the asked-for outcome.

**Bonus.** 1.1.3 also rejects `--expect=` on `--replace` (which
was always undocumented; docs say `--expect=` is for `--write`/
`--batch`, while `--replace` uses `--expect-N` / `--expect-1`).
1.1.2 silently accepted the misuse; 1.1.3 errors. Good cleanup.

## Wins

### dep-include path works end-to-end for pure-Cyrius mabda calls

**Setup.** `[deps.mabda]` git/tag/modules block in cyrius.cyml,
`cyrius deps --lock` to resolve + lock (5.7.7) or two-step
`cyrius deps && cyrius deps --lock` (5.7.8 workaround), single
mabda call in `src/main.cyr` (`gpu_err_name(GPU_ERR_NONE)`),
`cyrius build`, run.

**Result.** Build green (only sigil dead-code warnings — those
are unused-API noise, not errors), runs clean, prints expected
output. The whole pipeline — declare dep → resolve → symlink into
`lib/` → auto-include → compile → link → run — Just Works for
pure-Cyrius mabda functions. The friction we found (P1-P8) is
real but does not block this path.

### `cyim --write` heredoc handles em-dashes and unicode cleanly

The `main.cyr` body included an em-dash (`—`) inside a string
literal. `cyim --write <file>` from a heredoc round-tripped it
byte-for-byte. No escaping required.

## Verification log

### 2026-04-26 — swept against cyrius 5.7.8 + cyim 1.1.2 + owl 1.1.6

Each open item re-run with the exact repro from its section.
Each fix verified by the same repro now succeeding (or the help
text now containing the expected guidance).

- ✅ P4 fixed (count drift gone)
- ✅ owl head/tail idiom fixed (help text updated inline)
- ✅ cyim multi-edit fixed (`--batch` shipped)
- ❌ P1, P2, P3, P5 still open
- 🆕 P8 new in 5.7.8 (`--lock` regression)

### 2026-04-26 — re-swept against cyim 1.1.3

### 2026-04-26 — re-swept against cyrius 5.7.11

User confirmed 5.7.11 is current; remaining open items deferred
to a few releases out. All five open `cyrius deps` items re-run
with their original repros:

- ❌ P1 (modules-path validation) — still open. Re-added
  `[deps.hisab]` with non-existent `dist/hisab.cyr`; resolver
  reports `4 deps resolved` exit 0 and creates a dangling symlink
  to `~/.cyrius/deps/hisab/2.2.0/dist/hisab.cyr`. Same as 5.7.7 /
  5.7.8.
- ❌ P2 (`cyrius deps --help` runs resolver) — still open. Output:
  `3 deps resolved`, exit 0.
- ❌ P3 (deps not in top-level help) — still open. `cyrius help`
  output unchanged; only incidental hit is `vet` description
  containing the substring "dependencies".
- ❌ P5 (lockfile not auto-generated) — still open. Plain
  `cyrius deps` writes no `cyrius.lock`.
- ❌ P8 (`--lock` regression from cold) — still open. From clean
  slate, `cyrius deps --lock` reports `cyrius.lock: 0 deps locked`
  and writes an empty lockfile. Two-step workaround unchanged.

No new pain points caught in this sweep. cyim 1.1.3 and owl 1.1.6
unchanged.

- ✅ cyim multiple-`--expect=` resolved — now errors `cyim: duplicate flag: --expect` exit 2, file untouched. Atomic rejection, clear message. Better than the asked-for outcome.
- 📝 Bonus: 1.1.3 also rejects `--expect=` on `--replace` (which was always undocumented; docs say `--expect=` is for `--write`/`--batch` only). 1.1.2 silently accepted; 1.1.3 errors. Good cleanup.
- ❌ cyim `--grep` confirmed as literal substring match (not regex) — refined the note with crisper evidence (literal-`^` vs anchor-`^` probe).
- ✏️ P7 deferred per user feedback to mabda 3.0.1; mabda 3.0 is pure-Cyrius GPU-native pivot.
- 🎯 cyim `--grep` regex design agreed: ship `--find` (regex search) + `--regex=<flavor>` (flavor selector). User to land later.

### 2026-05-25 — toolchain pin bumped 5.7.11 → 6.0.1 (not yet re-swept)

Pin moved to cyrius `6.0.1` as part of the language + doc-standards
refresh. This is a **pin bump, not a verification sweep** — the open
`cyrius deps` items (P1 modules-path validation, P2 `--help` runs
resolver, P3 deps absent from top-level help, P5 lockfile not
auto-generated, P8 `--lock` cold regression) were **not** re-run
against 6.0.1. They are carried forward as-is. Next real sweep should
re-run each repro against 6.0.1 and reclassify; several may have been
addressed in the v6.0.0 stdlib/CLI clean-slate but that's unconfirmed
here. P7 remains deferred to mabda 3.0.1.

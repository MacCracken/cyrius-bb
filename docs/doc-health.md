---
name: cyrius-bb Documentation Health
description: Living state of doc currency in the cyrius-bb repo — fresh / stale / archive / open-question, refreshed as docs are touched
type: state
---

# Documentation Health — cyrius-bb

> **Last refresh**: 2026-09-26 (first full re-sweep since the 0.2.0 cut, at 0.8.3 + the `[Unreleased]` audio fix; every row re-verified against the tree. CONTRIBUTING.md's scaffold-era Project Structure, test path, toolchain pin and fmt-gate claim fixed; CLAUDE.md's Quick Start test command corrected; rows added for architecture notes 001 + 002; README, SECURITY, note 002, roadmap's v1.0 section and tooling-pain-points marked stale; the mabda open question closed; new open question on the passed v1.0 date). Previous refresh: 2026-05-25 (0.2.0 cut). | **Refresh cadence**: opportunistic — when a doc is touched, update its row; re-anchor the "Last refresh" date and the at-a-glance buckets when they drift.
>
> **Scope**: This repo only (`cyrius-bb`) — the whole `docs/` tree plus root-level files (README, CHANGELOG, CLAUDE.md, the required-root set, VERSION, cyrius.cyml, LICENSE). Upstream-dep docs (mabda, sankoch, sigil, shravan, kiran, impetus, the Cyrius stdlib) live in their own repos and are not audited here.
>
> **Why this exists early**: the first-party doc standard says doc-health is worth scaffolding "once a repo has more than ~30 docs or any meaningful drift surface; smaller repos can defer." cyrius-bb had only ~9 docs (13 in `docs/` at 0.8.3) and would normally defer — this ledger was created ahead of that threshold by request, to carry the convention from scaffold rather than retrofit it later. Structure mirrors [cyrius/docs/doc-health.md](https://github.com/MacCracken/cyrius/blob/main/docs/doc-health.md), scaled down for the small tree. Location is `docs/doc-health.md` (repo `docs/` root, **not** under `development/`) per [first-party-documentation.md § Development Docs](https://github.com/MacCracken/agnosticos/blob/main/docs/development/first-party/first-party-documentation.md) — the ledger sweeps the whole tree, so its location matches that scope.

This is a **ledger**, not a one-time audit. Rewrite-in-place as docs change.

---

## At a glance — 2026-09-26 inventory (0.8.3 + `[Unreleased]` audio fix)

**13 markdown files in `docs/` (this ledger included) + 9 root files.** Milestones M0–M6 have shipped; the pre-v1.0 console playtest is the open gate. Drift sits in docs the release passes skipped: README's and SECURITY's scaffold-era prose, note 002's 0.6.0-era size findings, and everything framed around the 2026-06-13 v1.0 date, which has passed. Bucket counts (21 rows; this ledger has none):

| Bucket | Count | What it means |
|---|---|---|
| ✅ **Fresh / touched this cycle** | 13 | CHANGELOG / CLAUDE.md / cyrius.cyml / VERSION / LICENSE / CONTRIBUTING / CODE_OF_CONDUCT + ADR README / template / 0003 + architecture README / note 001 + state.md. Accurate as of this sweep (CLAUDE.md + CONTRIBUTING fixed in it). |
| 🟡 **Stale — refresh in place** | 5 | README, SECURITY.md, note 002, roadmap.md (v1.0 section), tooling-pain-points.md — detail in each row. |
| 🟠 **Read-through outstanding** | 1 | `docs/design/breakout-date-verification.md` — open research item; its result now feeds the re-pinned v1.0 date. |
| 🔵 **Probably evergreen** | 2 | ADR 0001 (homage-from-observation) + ADR 0002 (original-assets-only) — load-bearing project principles; re-read at v1.0 close, not per release. |
| 📦 **Archive — frozen by design** | 0 | No archive tree yet. |
| ❓ **Open strategic question** | 2 | (1) `docs/design/` is outside the standard doc-layer map; (2) the v1.0 date (2026-06-13) has passed and needs a re-pin. See *Open questions* below. (The mabda manifest question from the prior sweep is closed.) |

Numbers roll up from the per-tier tables below.

---

## Tier 1 — Root + structural docs

| File | Last touched | Status | Action |
|---|---|---|---|
| `README.md` | 2026-05-25 | 🟡 Stale | Landing page — what / what-it-isn't / deps / build / status. **Stale since 0.2.0**: Status still reads "0.1.0 released; M1 (playable loop) on the dev tip" (0.8.3 is current; M0–M6 shipped); the Build block says 84 assertions (222 at 0.8.3, 253 with the `[Unreleased]` fix); Dependencies say sankoch + sigil stay commented out until M5 (wired at 0.6.0 via `[deps].stdlib`) and omit the vendored vani-core (audio, since 0.8.0). Refresh Status / Build / Dependencies in place. |
| `CHANGELOG.md` | 2026-09-26 | ✅ Fresh | **Source of truth per CLAUDE.md.** Newest cut `[0.8.3]` (2026-09-26: toolchain 6.6.6 + vani-core 1.2.5); `[Unreleased]` holds the audio fix (48 kHz S16 stereo, XRUN recovery), whose listening test is still pending. Keep a Changelog format. |
| `CLAUDE.md` | 2026-09-26 | ✅ Fresh | Durable preferences/process/procedures; version and other volatile state delegated to state.md — correct. **2026-09-26**: Quick Start test command `cyrius test src/test.cyr` → `cyrius test tests/cyrius-bb.tcyr` (the suite CI runs; the repo has never had a `src/test.cyr`). Its Work Loop step 2 and no-FFI rule still name mabda / kiran / impetus / shravan as the stack, which ADR 0003 deferred — owner's call whether to reword. |
| `cyrius.cyml` | 2026-09-26 | ✅ Fresh | Manifest + dep chain. Pin `6.6.6` (bumped 2026-09-26). `[deps].stdlib` carries sankoch + sigil (wired at M5, resolved from the toolchain snapshot) and their transitive modules, `sys` added at 0.8.3. The mabda / kiran / impetus / shravan / hisab blocks and the sankoch / sigil git fallbacks stay commented out (ADR 0003). |
| `VERSION` | 2026-09-26 | ✅ Fresh | Single source of truth: `0.8.3`, in sync with `cyrius.cyml` (`${file:VERSION}`) and the CHANGELOG header. The `[Unreleased]` audio fix carries no bump. |
| `LICENSE` | 2026-04-26 | ✅ Fresh | GPL-3.0-only — matches `cyrius.cyml` `license` field. |
| `CONTRIBUTING.md` | 2026-09-26 | ✅ Fresh | Contributor summary: the two ADR-driven hard constraints, the playtest-over-unit-test gate, the Cyrius conventions from CLAUDE.md. **2026-09-26**: Project Structure rewritten from the scaffold-era planned layout to the real modules; test entrypoint `src/test.cyr` → `tests/cyrius-bb.tcyr` (workflow + Adding a Module); the stale `6.2.2` pin dropped in favour of the `cyrius.cyml` pointer; the fmt line no longer claims a CI gate (CI runs build, lint and tests, not `cyrius fmt`). Like CLAUDE.md, Prerequisites still names mabda / kiran / impetus / shravan as the stack. |
| `SECURITY.md` | 2026-05-25 | 🟡 Stale | No-FFI / kavach-owns-sandbox framing still holds. **Stale since 0.2.0**: the save file (`~/.cyrius-bb/scores.cyb`) is listed as planned for M5 (shipped at 0.6.0: sankoch zlib + sigil HMAC-SHA256, tamper-rejecting), and `.ogg` user music as planned for M4 (never shipped — music was deferred, [note 001](architecture/001-no-ffi-audio.md); nothing in `src/` loads music). Audit History still says "scaffold stage (v0.1.0, no gameplay code)"; no audit has run yet (no `docs/audit/`). The upstream list names mabda / kiran / impetus / shravan (not deps, ADR 0003) and omits the vendored vani-core. |
| `CODE_OF_CONDUCT.md` | 2026-05-25 | ✅ Fresh | **Added 2026-05-25.** Contributor Covenant v2.1 pointer, cyrius gold-standard shape; enforcement contact `conduct@agnosticos.org`. |

**Required-root set complete** (resolved 2026-05-25): `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` were drafted by hand this pass rather than via `cyrius init` re-scaffold. The standard prefers tooling-scaffolded root files — if/when `cyrius init` is re-run or a template propagation lands, reconcile these hand-written versions against the canonical templates.

---

## Tier 2 — ADRs (`docs/adr/`)

3 ADRs. Re-read pass at v1.0 close; ADRs document decisions, not status.

| File | Last touched | Status | Notes |
|---|---|---|---|
| `README.md` | 2026-05-25 | ✅ Fresh | Index + conventions. One-line hook per ADR; matches the 3-ADR set. |
| `template.md` | 2026-04-26 | ✅ Fresh | Copy-to-start template. |
| `0001-homage-from-observation.md` | 2026-04-26 | 🔵 Evergreen | Mechanics traced to documented public sources; no ROM/binary reverse-engineering. Foundational. |
| `0002-original-assets-only.md` | 2026-04-26 | 🔵 Evergreen | New art/audio/level data or public-domain reference only; no Atari-era assets. Foundational. Its audio line says the SFX are "synthesized via shravan"; they are self-rolled instead ([note 001](architecture/001-no-ffi-audio.md)). The policy itself holds. |
| `0003-self-rolled-primitives.md` | 2026-05-25 | ✅ Fresh | **Added 2026-05-25.** Defer kiran/impetus/mabda/soorat; tool primitives on bare stdlib (cyrius-doom pattern). Its plan to uncomment the sankoch / sigil git deps at M5 played out differently: M5 (0.6.0) wired them through `[deps].stdlib` from the toolchain snapshot. Re-read at v1.0 close. |

---

## Tier 3 — Architecture (`docs/architecture/`)

| File | Last touched | Status | Action |
|---|---|---|---|
| `README.md` | 2026-09-26 | ✅ Fresh | Index of the numbered notes (001, 002); never renumber. |
| `001-no-ffi-audio.md` | 2026-09-26 | ✅ Fresh | Why audio is self-rolled square-wave PCM played through the vendored vani-core ALSA shim, and why there is no music. **2026-09-26**: refreshed for the audio fix — device-native 48 kHz S16 stereo, the XRUN-recovering stream model, two console gotchas (the codec's mixer, a busy card). |
| `002-save-deps-binary-size.md` | 2026-05-26 | 🟡 Stale | Why wiring sankoch + sigil at M5 ~4x'd the binary. **Its measurements are from 0.6.0 on toolchain 6.0.1** (117,184 → 454,352 B, and "`CYRIUS_DCE=1` produces the same size as a non-DCE build"). On 6.6.6 DCE does trim: 0.8.3 builds 2,068,920 B plain vs 999,864 B with DCE, and calling `zlib_*` directly instead of the `compress` / `decompress` dispatchers measured 778,680 B (−22%; state.md Next item 3). Re-measure and update in place. |

---

## Tier 4 — Development (`docs/development/`)

> state.md + roadmap.md are the canonical operational surface — they rotate every release/milestone. tooling-pain-points.md is an append-only dogfood log.

| File | Last touched | Status | Action |
|---|---|---|---|
| `state.md` | 2026-09-26 | ✅ Fresh | **Rotates every release.** Current through 0.8.3 plus the `[Unreleased]` audio fix (253 assertions, DCE 999,928 B; listening test pending). Its v1.0 target (2026-06-13, "comfortably ahead") has passed — see Open question 2. |
| `roadmap.md` | 2026-09-26 | 🟡 Stale | Milestone sequence M0→v1.0; M0–M6 marked shipped, M4's carried-forward note updated 2026-09-26 for the audio fix. **The v1.0 section is still built on the 2026-06-13 ship date** ("50 days from scaffold", "ships 8 days before the Beat 1 solstice demo"); that day shipped 0.7.2 instead. Rewrite the section once the date is re-pinned (Open question 2). |
| `tooling-pain-points.md` | 2026-05-25 | 🟡 Stale | Append-only dogfood log (cyrius/cyim/owl). Its header still reads "Pin now at cyrius 6.0.1"; the pin is `6.6.6`, and the open `cyrius deps` items (P1/P2/P3/P5/P8) have not been re-swept since cyrius 5.7.11. Next real sweep should re-run each repro and reclassify. |

---

## Tier 5 — Design (`docs/design/`)

> Project-specific subtree (art direction, asset pipeline, palette/audio sourcing) — **not** in the standard doc-layer map. See *Open questions* #1.

| File | Last touched | Status | Action |
|---|---|---|---|
| `breakout-date-verification.md` | 2026-04-26 | 🟠 Read-through | **Open action item.** Verify Breakout's original release date (April 13, 1976 is most-cited but contested) against primary sources before pinning it in any launch collateral. The v1.0 date it framed (2026-06-13) has passed, so the result now feeds the re-pinned date (Open question 2). |

---

## Open questions

Strategic doc-tree questions — not stale rows, but decisions that want an owner.

1. **`docs/design/` is outside the standard doc-layer map.** It's referenced from CLAUDE.md as a deliberate project subtree, but the map has no "design" layer. Its one current file is really a source-verification action item (closer to `docs/sources.md` territory). The decision was due at M2/M3 (the art pass) and is still open at 0.8.3: does `docs/design/` earn its place as a project-specific subtree, or do its contents migrate to standard layers? Inventing a top-level `docs/` subdir is an ADR-worthy call per the standard.
2. **The v1.0 date has passed.** [roadmap.md](development/roadmap.md) pinned v1.0 to 2026-06-13 (50 years + 2 months after Breakout's most-cited release date); that day shipped 0.7.2 instead. v1.0 is gated on the console playtest (state.md Next item 1). roadmap.md's v1.0 section, state.md (Milestones + Next item 2) and the design note still frame the old date. Re-pinning is the coordinator's call (the design note lists the framing options); then update those three docs.

**Resolved 2026-05-25**:
- The three required root files (`CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`) were missing; drafted by hand and added. Caveat in Tier 1: reconcile against canonical `cyrius init` templates if a re-scaffold lands.
- The prior "mabda/sankoch/sigil possibly folded into stdlib" question is **answered by [ADR 0003](adr/0003-self-rolled-primitives.md)**: cyrius-bb self-rolls rendering and does not depend on mabda regardless of whether it folded; sankoch + sigil are retained as git deps for the save file. (The narrower manifest-cleanup follow-up is closed below.)

**Closed 2026-09-26** (resolved earlier; the ledger had not recorded it):
- The former open question 2, **`cyrius.cyml` still listing mabda as an active dep**. mabda was commented out later on 2026-05-25, per [ADR 0003](adr/0003-self-rolled-primitives.md), and sankoch + sigil came back at M5 through `[deps].stdlib` rather than as git deps. The build's only warnings are now the two upstream-sigil ones listed in CHANGELOG 0.8.3, plus the static-data advisory.

---

## Refresh procedure

When docs are touched:

1. Find the affected row in the relevant tier table.
2. Update **Last touched** to the new date.
3. Update **Status** if the bucket changed.
4. Update **Action** if the next step changed.
5. If a doc moved or was archived, update its row.
6. Re-anchor "Last refresh" in the header.

When the at-a-glance bucket counts drift by more than ~2 in any cell, refresh that table. This file's cadence is **opportunistic** (touched when other docs are touched), not periodic.

---

## What this file is NOT

- Not a substitute for [`development/state.md`](development/state.md) (live code/version/milestone state).
- Not a CHANGELOG (which records what shipped, not what's stale).
- Not a TODO list (open work lives in [`development/roadmap.md`](development/roadmap.md)).
- Not a per-doc review log (this is where each doc *stands*, not the per-doc reasoning).

---

*Initial scaffold: 2026-05-25 (v0.1.0) — created during the 6.0.1 language + doc-standards refresh. Carried the doc-health convention from agnosticos' first-party standard ahead of the ~30-doc threshold by request. Refresh in place when docs are touched.*

---
name: cyrius-bb Documentation Health
description: Living state of doc currency in the cyrius-bb repo — fresh / stale / archive / open-question, refreshed as docs are touched
type: state
---

# Documentation Health — cyrius-bb

> **Last refresh**: 2026-05-25 (language + doc-standards refresh: toolchain pin 5.7.11 → 6.0.1; stale `docs/development/applications/` → `first-party/` standards links fixed in CLAUDE.md + roadmap.md; this ledger scaffolded; the three missing required root files — CONTRIBUTING / SECURITY / CODE_OF_CONDUCT — drafted and added). | **Refresh cadence**: opportunistic — when a doc is touched, update its row; re-anchor the "Last refresh" date and the at-a-glance buckets when they drift.
>
> **Scope**: This repo only (`cyrius-bb`) — the whole `docs/` tree plus root-level files (README, CHANGELOG, CLAUDE.md, the required-root set, VERSION, cyrius.cyml, LICENSE). Upstream-dep docs (mabda, sankoch, sigil, shravan, kiran, impetus, the Cyrius stdlib) live in their own repos and are not audited here.
>
> **Why this exists early**: the first-party doc standard says doc-health is worth scaffolding "once a repo has more than ~30 docs or any meaningful drift surface; smaller repos can defer." cyrius-bb has only ~9 docs and would normally defer — this ledger was created ahead of that threshold by request, to carry the convention from scaffold rather than retrofit it later. Structure mirrors [cyrius/docs/doc-health.md](https://github.com/MacCracken/cyrius/blob/main/docs/doc-health.md), scaled down for the small tree. Location is `docs/doc-health.md` (repo `docs/` root, **not** under `development/`) per [first-party-documentation.md § Development Docs](https://github.com/MacCracken/agnosticos/blob/main/docs/development/first-party/first-party-documentation.md) — the ledger sweeps the whole tree, so its location matches that scope.

This is a **ledger**, not a one-time audit. Rewrite-in-place as docs change.

---

## At a glance — 2026-05-25 inventory (scaffold stage, v0.1.0)

**9 markdown files in `docs/` + 9 root files** (the required-root set is now complete). The project is still at scaffold (no gameplay code yet), so almost everything is recently written and accurate; the live drift surface is small. Bucket counts:

| Bucket | Count | What it means |
|---|---|---|
| ✅ **Fresh / touched this cycle** | 14 | README / CHANGELOG / CLAUDE.md / state.md / roadmap.md / cyrius.cyml / tooling-pain-points.md + the 3 new required-root files (CONTRIBUTING / SECURITY / CODE_OF_CONDUCT, added 2026-05-25) + ADR README/template / architecture README index. (8 root/dev files touched 2026-05-25 in the 6.0.1 + standards-link + root-file pass.) |
| 🟡 **Stale — refresh in place** | 0 | None. (tooling-pain-points.md carries *open-but-not-re-swept* items against 6.0.1 — flagged in its own row, not stale prose.) |
| 🟠 **Read-through outstanding** | 1 | `docs/design/breakout-date-verification.md` — open research action item; resolve before any launch collateral pins the April-13-1976 date. |
| 🔵 **Probably evergreen** | 2 | ADR 0001 (homage-from-observation) + ADR 0002 (original-assets-only) — load-bearing project principles; re-read at v1.0 close, not per release. |
| 📦 **Archive — frozen by design** | 0 | No archive tree yet. |
| ❓ **Open strategic question** | 2 | (1) `docs/design/` is outside the standard doc-layer map; (2) mabda/sankoch/sigil now appear in the Cyrius stdlib — possible dep simplification. See *Open questions* below. (The missing-root-files question from the prior sweep was resolved 2026-05-25.) |

Numbers roll up from the per-tier tables below.

---

## Tier 1 — Root + structural docs

| File | Last touched | Status | Action |
|---|---|---|---|
| `README.md` | 2026-04-26 | ✅ Fresh | Landing page — what / what-it-isn't / deps / build / status. Accurate for scaffold stage. Refresh the Status block at each milestone. |
| `CHANGELOG.md` | 2026-04-26 | ✅ Fresh | **Source of truth per CLAUDE.md.** Only `[0.1.0]` so far; `[Unreleased]` empty. Keep a Changelog format. |
| `CLAUDE.md` | 2026-05-25 | ✅ Fresh | Durable preferences/process/procedures. **2026-05-25**: standards links repointed `applications/` → `first-party/`; shared-crates link + doc-health pointer added. Version not inlined (delegated to state.md) — correct. |
| `cyrius.cyml` | 2026-05-25 | ✅ Fresh | Manifest + dep chain. **2026-05-25**: toolchain pin `5.7.11` → `6.0.1`. All 16 stdlib modules resolve against 6.0.1; pending deps (kiran/impetus/shravan/hisab) still commented out awaiting upstream dist bundles. |
| `VERSION` | 2026-04-26 | ✅ Fresh | Single source of truth (`0.1.0`). Bumped at milestone close per CLAUDE.md. |
| `LICENSE` | 2026-04-26 | ✅ Fresh | GPL-3.0-only — matches `cyrius.cyml` `license` field. |
| `CONTRIBUTING.md` | 2026-05-25 | ✅ Fresh | **Added 2026-05-25.** Adapted from the mabda sibling; leads with the two ADR-driven hard constraints (original-assets, homage-from-observation), the playtest-over-unit-test gate, and the Cyrius conventions from CLAUDE.md. |
| `SECURITY.md` | 2026-05-25 | ✅ Fresh | **Added 2026-05-25.** No-FFI / kavach-owns-sandbox framing; the save file (`scores.cyb`, sankoch + sigil) is the named untrusted-input surface. Pre-1.0 supported-versions table; audit history empty (honest — no audit run yet). |
| `CODE_OF_CONDUCT.md` | 2026-05-25 | ✅ Fresh | **Added 2026-05-25.** Contributor Covenant v2.1 pointer, cyrius gold-standard shape; enforcement contact `conduct@agnosticos.org`. |

**Required-root set complete** (resolved 2026-05-25): `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md` were drafted by hand this pass rather than via `cyrius init` re-scaffold. The standard prefers tooling-scaffolded root files — if/when `cyrius init` is re-run or a template propagation lands, reconcile these hand-written versions against the canonical templates.

---

## Tier 2 — ADRs (`docs/adr/`)

2 ADRs. Re-read pass at v1.0 close; ADRs document decisions, not status.

| File | Last touched | Status | Notes |
|---|---|---|---|
| `README.md` | 2026-04-26 | ✅ Fresh | Index + conventions. One-line hook per ADR; matches the 2-ADR set. |
| `template.md` | 2026-04-26 | ✅ Fresh | Copy-to-start template. |
| `0001-homage-from-observation.md` | 2026-04-26 | 🔵 Evergreen | Mechanics traced to documented public sources; no ROM/binary reverse-engineering. Foundational. |
| `0002-original-assets-only.md` | 2026-04-26 | 🔵 Evergreen | New art/audio/level data or public-domain reference only; no Atari-era assets. Foundational. |

---

## Tier 3 — Architecture (`docs/architecture/`)

| File | Last touched | Status | Action |
|---|---|---|---|
| `README.md` | 2026-04-26 | ✅ Fresh | Index only — currently empty (no invariants surfaced yet). First numbered note (`NNN-*.md`) likely lands at M1: paddle-english tuning or 2.5D depth-layer ordering. |

---

## Tier 4 — Development (`docs/development/`)

> state.md + roadmap.md are the canonical operational surface — they rotate every release/milestone. tooling-pain-points.md is an append-only dogfood log.

| File | Last touched | Status | Action |
|---|---|---|---|
| `state.md` | 2026-05-25 | ✅ Fresh | **Rotates every release.** **2026-05-25**: Cyrius pin updated to 6.0.1. Still reflects scaffold stage (no gameplay code). |
| `roadmap.md` | 2026-05-25 | ✅ Fresh | Milestone sequence M0→v1.0, v1.0 pinned 2026-06-13. **2026-05-25**: stale `applications/` standards link fixed. |
| `tooling-pain-points.md` | 2026-05-25 | ✅ Fresh | Append-only dogfood log (cyrius/cyim/owl). **2026-05-25**: pin-bump entry added — open `cyrius deps` items (P1/P2/P3/P5/P8) **not yet re-swept against 6.0.1**, carried forward. Next real sweep should re-run each repro and reclassify. |

---

## Tier 5 — Design (`docs/design/`)

> Project-specific subtree (art direction, asset pipeline, palette/audio sourcing) — **not** in the standard doc-layer map. See *Open questions* #2.

| File | Last touched | Status | Action |
|---|---|---|---|
| `breakout-date-verification.md` | 2026-04-26 | 🟠 Read-through | **Open action item.** Verify Breakout's original release date (April 13, 1976 is most-cited but contested) against primary sources before pinning it in any launch collateral. Resolution feeds the v1.0 ship-date framing in roadmap.md. |

---

## Open questions

Strategic doc-tree questions — not stale rows, but decisions that want an owner.

1. **`docs/design/` is outside the standard doc-layer map.** It's referenced from CLAUDE.md as a deliberate project subtree, but the map has no "design" layer. Its one current file is really a source-verification action item (closer to `docs/sources.md` territory). Decide at M2/M3 (art pass) whether `docs/design/` earns its place as a project-specific subtree or its contents migrate to standard layers. Inventing a top-level `docs/` subdir is an ADR-worthy call per the standard.
2. **mabda / sankoch / sigil now ship in the Cyrius stdlib** (present in `cyrius/lib/` at 6.0.1) but are declared here as pinned git deps. May be a dep-graph simplification once confirmed they're genuinely folded (vs vendored for cyrius's own build). Revisit when wiring real rendering/save code at M1/M5 — verify against the live [shared-crates registry](https://github.com/MacCracken/agnosticos/blob/main/docs/development/planning/shared-crates.md) before changing the manifest.

**Resolved 2026-05-25**: the three required root files (`CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`) were missing; drafted by hand and added this pass. Caveat carried in Tier 1: the standard prefers `cyrius init`-scaffolded root files — reconcile against the canonical templates if a re-scaffold lands.

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

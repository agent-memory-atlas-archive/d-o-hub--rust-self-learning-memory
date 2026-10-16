# Project Status — Self-Learning Memory System

**Last Updated**: 2026-10-16
**Released Version**: v0.1.45 (tag `8ee711bd`, GitHub Release 2026-10-16) — **not on crates.io** (newest published `0.1.34`)
**Workspace Version**: 0.1.46 (post-v0.1.45 bump, landed with the closeout; tag `v0.1.46` is the publish vehicle)
**Edition**: Rust 2024
**Active plan**: **release closeout** (`GOAP_RELEASE_CLOSEOUT_2026-10-16.md`) — the audit-backlog wave is merged and
closed (`GOAP_AUDIT_BACKLOG_WAVE_2026-10-04.md`); what remains is the false-green crates.io publish (#1179), the
red Turso nightly (#1180), the readiness-gate run-ordering fix (#1177), guardrail docs (#1175) and the `0.1.46`
bump. Follow-up issues #1068, #1089 and #1076 stay queued behind it.
**Branch**: `main` @ `8ee711bd`

## Open tracker (live)

| Kind | Items |
|------|--------|
| Open PRs | live list — run `gh pr list --state open`; the release-closeout queue merges serially (order in the active plan under `plans/`), then the post-release bump PR |
| Open issues | 5 open (run `gh issue list --state open` — #1068, #1076, #1089, #1109, #1165) — per-issue verdicts, evidence and queue position live in `STATUS/GAP_ANALYSIS_LATEST.md` |
| Open Dependabot alerts | 12 (2 high, 2 moderate, 8 low) — high/moderate fixed on `main` via #1174; the 8 low need `libsql 0.10.0` |

## Corrected claim (2026-10-04)

The "P0 plan gaps: **0 open code-side**" line once below this section described the **CI-trust / attribution** campaign (ADR-079,
CIT-A1…A5, PTA, RAT) and is still true for that campaign. It was never true of the repository as a whole: the
22-issue audit backlog was open the entire time and absent from the trackers, which is how "0 open gaps" got
restated at each refresh. Read any campaign claim as campaign-scoped; the repo-wide register is
`STATUS/GAP_ANALYSIS_LATEST.md`.

## Recent completed (2026-10-05 — audit wave W1–W4 + hygiene)

| Wave | Result |
|------|--------|
| W1 #1077 | ✅ #1131 (`874df209`) — ranking snapshot cloned; read guard dropped before every `.await`; contention test |
| W2 #1086 | ✅ #1135 (`75007b51`) — monotonic `Instant`; sub-second MCP latency no longer truncates to 0 ms |
| W3 #1088 | ✅ #1134 (`347b3296`) — Turso tag transactions roll back on every failure path |
| W4 #1066 | ✅ #1139 (`f1c31699`, `0485bb66`) — redb episode→session index ranks by `(timestamp, session_id)`; open-time repair heals stale write-order winners (no schema bump) |
| Hygiene | ✅ #1128 docs-integrity false positives, #1129 tracker reconciliation, #1133 local hook sensors, #1136 dependabot actions bump |

## Recently merged (2026-10-09…12)

| Slice | PR | State |
|-------|-----|-------|
| W5 #1075 query-aware lexical fallback | #1138 | ✅ merged |
| W6 #1085 health probes + URL redaction (E1) | #1140 | ✅ merged |
| #1067 / #1064 / #1070 / #1063 / #1071 | #1149, #1147, #1150, #1146, #1145 | ✅ merged |
| #1073 / #1078 / #1074 | #1151, #1154, #1160 | ✅ merged |
| #1087 capability truth | #1144, #1152, #1158 | ✅ merged |
| #1092 lint slice 2 | #1153 | ✅ merged as `b6326d90`; crate-root allows 317 → 0 |
| Nightly JSON / fuzz toolchain fixes | #1159, #1161 | ✅ merged |
| Release-plan refresh | #1141 | ✅ merged |

## Recent completed (2026-10-13…16 — v0.1.45 ship + closeout)

| Slice | PR / commit | State |
|-------|-------------|-------|
| v0.1.45 release | `8ee711bd` → tag `v0.1.45` | ✅ shipped via `release-manager.sh ship --execute` (never `gh release create`) |
| rustls + quinn-proto advisories | #1174 (`2d5ed2b0`) | ✅ merged — cleared the 2 high + 2 moderate alerts and unblocked `main` |
| Release docs + version bump | #1169 | ✅ merged as `8ee711bd` |
| crates.io publish false-green | #1179 | 🔄 in review — `if: ${{ !inputs.dry-run … }}` string-coercion skipped every real upload since `493cfb01`; adds `Verify publish landed` + gate-contract/fixture sensors |
| Readiness gate run ordering | #1177 | 🔄 in review — count the latest check run per name (superseded runs were reported as current) |
| Guardrail docs (advisory/ruleset/drift traps) | #1175 | 🔄 in review — owns `AGENTS.md` + `agent_docs/LESSONS.md` |
| Turso nightly: 36 failures → 4 real defects | #1180 | 🔄 in review — metrics column/unit drift, multi-dimension read gap, pooled-checkout misuse, error-swallowing asserts; `required-features` + feature-specific nightly step |

## Recent completed (2026-10-04 — release pipeline C5–C7)

| Wave | Result |
|------|--------|
| Release pipeline C5–C7 (#1109) | ✅ PR #1125 (`4fb0e20f`) — "Verify a release" book page + SECURITY.md signer-pinned attestation commands, `release.yml` PR trigger removed (it ran only skipped jobs), `v`-prefixed tag glob, publish-path `persist-credentials: false`, `host` OIDC blanked on SBOM build steps |
| Audit-backlog reconciliation | ✅ #1129 (`26b12fc8`) — 22 issues re-validated at `74a44a15`, registered in `GAP_ANALYSIS_LATEST.md`, wave slices W1–W6 queued (ACT-370…375) |

## Recent completed (2026-10-02 — v0.1.44 shipped)

| Wave | Result |
|------|--------|
| Pattern-search input bounds (#1113) | ✅ `search_patterns`/`recommend_patterns` clamp `limit`/`min_relevance` and truncate oversized `query`/`task_description`/`domain`/`tags` (CWE-770) |
| Architecture refresh (#1094 → #1110) | ✅ architecture/serialization/status evidence aligned with v0.1.44; bincode references corrected to postcard |
| Coverage floor (#1090 → #1117) | ✅ 70% is the blocking floor, 90% the aspirational target, with comparator unit tests |
| v0.1.44 release | ✅ tag `v0.1.44` on `4f4f4ba8`; GitHub Release with dist artifacts; drift issue auto-closed; workspace bumped to 0.1.45 |

## Recent completed (pointer — 2026-07…2026-10-01)

| Wave | Result |
|------|--------|
| Release-pipeline proposal (#1109) | ✅ code complete: C3/C4 (#1121 draft-first + attestations), C1/C2 (#1123 OIDC-only publish + dead-trigger fix), C5–C7 (#1125); manual crates.io steps tracked in ACT-368 |
| v0.1.43 / LOC ceiling | ✅ 2026-10-01 — tag `v0.1.43`; `check-loc.sh` now blocking in File Structure Validation (#1103) |
| MCP OAuth 2.1 + trusted rate-limit identity (#1082/#1084) | ✅ #1101 |
| redb fail-closed schema (#1069), embedding activation identity (#1072), MCP `tools/list` (#1083), pool ownership (#1060–#1062), MCP docs (#1099) | ✅ #1095–#1100 |
| Retrieval judgment (#1030 → #1041) + rerank (#1031 → #1042) + merge tooling (#1046) | ✅ 2026-09-24 |
| Feedback-to-ranking (ADR-082) + ADR registry | ✅ #952 (2026-08-13); lifecycle `Proposed` |
| CI trust + attribution closure (ADR-079/CIT/PTA/RAT) | ✅ #916/#927/#930/#938/#940/#947 (2026-08-06…12); stage 4 proof maintainer-external |
| v0.1.36…v0.1.42 ships + post-bumps + skills + R-F8/R-F9 | ✅ 2026-07…09 |

## Snapshot (current)

| Area | State |
|------|--------|
| First-party merge gate | ✅ **Live** — ruleset `9591004` requires `Codacy Static Code Analysis` + `CI / Required` (strict up-to-date policy); the aggregate is causally same-run and rejects `skipped`/`cancelled`/timed-out results |
| Repo-wide open issues | **5 open**: #1068, #1076, #1089 (audit follow-ups), #1109 (manual crates.io registrations, maintainer-external), #1165 (v0.1.45 drift — closes with the `0.1.46` bump) — register: `STATUS/GAP_ANALYSIS_LATEST.md` |
| Merged since 2026-10-06 | #1130, #1142, #1143, #1148, #1138, #1140, #1144, #1145, #1146, #1147, #1149, #1150, #1151, #1152, #1154, #1158, #1159, #1160, #1161, #1153, #1174 |
| Production pattern query relevance | ✅ **#1074 merged in #1160** — pattern operations use one runtime provider snapshot |
| `episodes.created_at` watermark | ✅ **#1067 merged in #1149** — durable modification watermark and bounded keyset pages |
| Release | `v0.1.45` tagged `8ee711bd` and published on GitHub 2026-10-16 |
| crates.io | ⚠️ **Stalled at `0.1.34`** (`do-memory-mcp` at `0.1.31`) — the publish step was skipped by a string-coerced guard while the job stayed green; #1179 fixes it and `0.1.46` is the first tag that can publish |
| Nightly (Turso native) | ⚠️ **Red since 2026-10-13** (36 of 106) — repaired in #1180; `86 + 20 = 106`, 0 failures, nothing ignored or waived |
| Code execution | Fail-closed; S1.1c Wasmtime/WASI **NO-GO** |
| Serialization | Postcard required |
| Skill evals / routes | 40/40 |

## Immediate priorities

| Priority | Item | ID | Status |
|----------|------|-----|--------|
| P0 | Land the closeout PR **first** — it carries the `0.1.46` bump, which clears critical drift for the whole queue so no PR needs the `release-preparation` override | ACT-402 | 🔄 next |
| P0 | Then merge the rest serially with full green CI (Codacy included, no waivers): #1179 → #1177 → #1175 → #1180 | ACT-399…401 | 🔄 |
| P0 | Cut tag `v0.1.46` after those land and verify every crate resolves on crates.io | ACT-409 | ⏳ |
| P0 | `0.1.46` bump PR → tag `v0.1.46` → verify every crate resolves on crates.io **and** `Verify publish landed` green | ACT-402 | ⏳ |
| P0 | Next nightly run green (scheduled + dispatch), including the new feature-specific step | ACT-403 | ⏳ |
| P1 | `libsql 0.9.30 → 0.10.0` PR (8 low alerts); record `lru` as upstream-blocked by `augurs-changepoint` → `rv 0.16` | ACT-404 | ⏳ |
| P1 | Sensors so this class of bug fails locally: schema-vs-SQL column check; LESSON + AGENTS trap entries for `required-features` vs `cfg_attr(…, ignore)` and error-swallowing asserts | ACT-406, ACT-407 | ⏳ |
| P2 | Decide whether scheduled/tag nightly runs gate releases (`do_ci_check` `.event` filtering) — hardening only, never weakened at ship time | ACT-405 | ⏳ |
| P2 | Follow-up watermark sync improvements (#1068, #1089) | S09/M08 | ⏳ post-closeout |
| P2 | Follow-up embedding reuse and batching (#1076) | R05 | ⏳ post-closeout |
| P0 | ADR-079 stage 4 live fault-injection proof; ADR-080/081 lifecycle; #1109 crates.io registrations + secret deletion | maintainer | ⏸ external |
| P2 | Research/product spikes (R-F1…R-F7, R-F10) | R-F* | ⏸ DEFER |

## Canonical companions

- Roadmap: `plans/ROADMAPS/ROADMAP_ACTIVE.md`
- Goals / actions / GOAP: `plans/GOALS.md`, `plans/ACTIONS.md`, `plans/GOAP_STATE.md`
- Active closeout: `plans/GOAP_RELEASE_CLOSEOUT_2026-10-16.md`
- Gaps: `plans/STATUS/GAP_ANALYSIS_LATEST.md`
- Validation: `plans/STATUS/VALIDATION_LATEST.md`
- Archive: `plans/archive/2026-07-consolidation/`

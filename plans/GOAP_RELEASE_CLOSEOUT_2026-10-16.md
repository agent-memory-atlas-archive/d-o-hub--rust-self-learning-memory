# GOAP — Release Closeout 2026-10-16 (v0.1.45 shipped, publish stalled, nightly red)

- **Last Updated**: 2026-10-16
- **Workspace**: `0.1.45` · **Latest tag**: `v0.1.45` (GitHub Release published 2026-10-16T10:30:09Z, tag SHA `8ee711bd`)
- **Scope of this doc**: the three things the v0.1.45 ship left behind — a **false-green crates.io publish**,
  a **5-day-red nightly**, and a **merge queue that the release-drift gate blocks**. It is the handoff record for
  whoever continues the chain; every claim below has a command next to it.
- **Prior plan**: `GOAP_AUDIT_BACKLOG_WAVE_2026-10-04.md` (merged closed); `GOAP_RELEASE_PIPELINE_C5_C7_2026-10-04.md`.

---

## 1. Situation (verified, not asserted)

| Fact | Evidence |
|------|----------|
| `v0.1.45` is released on GitHub | `gh release list` → `v0.1.45  2026-10-16T10:30:09Z`; tag → `8ee711bd` |
| …but **nothing was published to crates.io** | `do-memory-core`/`-storage-redb`/`-storage-turso` newest `0.1.34`, `do-memory-mcp` newest `0.1.31`; `v0.1.44` and `v0.1.45` never appeared |
| The publish run reported **success** | `gh run list --workflow release.yml` → `38883104363  8ee711bd  success  push`; the dispatched `publish-crates.yml` run `38887674688` shows `Publish to crates.io` with `conclusion=skipped;duration_ms=0` |
| Nightly has been red since 2026-10-13 | `gh run list --workflow nightly-tests.yml --limit 5` → five consecutive `failure` (schedule + push) |
| Pre-fix baseline is exactly **36 failures / 106 run** | run `38883104337` job log: `Summary [3.200s] 106 tests run: 70 passed, 36 failed, 514 skipped` |
| The drift gate now reds **every** PR | `Check for unreleased changes` → `severity=critical reason=version_not_advanced` (`workspace_version=0.1.45`, `latest_tag=v0.1.45`) |
| 12 Dependabot alerts open (2 high, 2 moderate, 8 low) | `gh api repos/:owner/:repo/dependabot/alerts?state=open` — all transitive (libsql chain, `augurs-changepoint` → `rv`/`lru`) |
| Advisory fixes for the 2 high + 2 moderate are merged | #1174 (`2d5ed2b0`) on `main`; the remaining 8 low need `libsql 0.9.30 → 0.10.0` |

## 2. Merge pipeline (order is forced, not preferred)

Ruleset `main-protection` has `strict_required_status_checks_policy`, so **merges are serial** and each merge
invalidates every other PR's required runs. Blocking graph:

```
#1179 publish guard   ─┐  (must be in main before the tag that publishes)
#1177 readiness gate  ─┤  (latest-run-per-name; makes the drift re-run readable)
#1175 guardrail docs  ─┤  (owns AGENTS.md + agent_docs/LESSONS.md — nothing else may touch them)
#1180 nightly Turso   ─┤  (fixes the red nightly; owns memory-storage-turso/*)
docs closeout (this)  ─┘  (plans/ only — no file overlap with the above)
        │
        ▼
0.1.46 bump PR  ──► tag v0.1.46 ──► release.yml ──► publish-crates.yml (first real publish)
```

Why the bump PR is the only publish vehicle: `release.yml` dispatches `publish-crates.yml --ref <tag>`, so both
the workflow **and** `.github/actions/publish-crate` come from the tag. `v0.1.45` @ `8ee711bd` carries the broken
guard, and `Verify tag matches version` refuses a tag that does not equal the manifest. **No retag, no manual
`cargo publish`, no static token** (OIDC-only per ADR-078; tag-only authority per ADR-072).

`0.1.46` will publish `do-memory-mcp` for the first time since `0.1.31`, and everything since `0.1.34`.

## 3. Nightly Turso triage — four root causes (#1180)

The red suite was not flakiness. `--run-ignored only` force-executes the `#[ignore]`d Turso tests, and the
assertions printed a fixed sentence instead of the `Err`, so four independent defects were reported as one
undifferentiated "36 tests failed".

| RC | Symptom | Actual defect | Fix |
|----|---------|---------------|-----|
| **RC4** | `monitoring_capacity_tests` red | `store_agent_metrics`/`store_task_metrics` INSERT+SELECT named `agent_metrics.last_execution_time` and `task_metrics.avg_completion_time`; the DDL creates `last_execution` and `avg_completion_time_ms`. **Every Turso metrics write failed**, on every fresh database. `avg_duration` was written `as_secs_f64()` into an `*_ms` column and read back `from_secs_f64(ms/1000)`. | `11f1d628` — declared column names, `i64` milliseconds round-trip |
| **RC1b** | `multi_dimension_routing`, `phase1_validation` red (`no such table: embeddings`) | Writes are dimension-aware under `turso_multi_dimension`, and that build never creates the legacy `embeddings` table — but `_get_embedding_internal` (plus the batch reader that delegates to it) and `has_vector_embeddings` still queried it. | `62639fe3` — reads mirror the write path; count spans the four indexed dimension tables (`embeddings_other` has no vector index) |
| **RC3** | pooled-mode-only failures | `get_connection_with_id()` errors in direct-connection mode only; capacity intents, sync watermark, embedding listing and the modified-since episode query used it instead of a scoped checkout. | `2a98b60a` — `with_connection`/`with_connection_with_id` so the permit covers the whole closure |
| **RC1 / RC2** | `vector index(insert): dimensions are different: 1536 != 384`, plus uninformative asserts | Fixtures stored 2- and 5-element vectors into an `F32_BLOB(384)` column (rejected by the vector index before the storage layer). Asserts mapped `Result` → bare `bool`, hiding the cause — this is **why three production bugs stayed hidden while CI was red**. | `d64c84d9` — schema-valid fixtures, asserts print `{result:?}` |
| **CI config** | feature-gated tests "passed" for the wrong reason | Per-test `cfg_attr(not(feature = …), ignore)` cannot protect a suite from `--run-ignored only`, which force-runs them against the default schema. | `50414816` — `[[test]] required-features` + a nightly step that runs both suites **with** the feature |

The new nightly step uses `--run-ignored all`, **not** `only`: with the feature enabled those tests are no longer
ignored, and `only` would filter all of them out — a silent no-op that would have looked green.

**Verification (local, exact nightly invocations — nothing ignored, deleted or waived):**

```
cargo nextest run -p do-memory-storage-turso --run-ignored only
  → 86 tests run: 86 passed, 514 skipped            (default schema)
cargo nextest run -p do-memory-storage-turso --features turso_multi_dimension \
  --test multi_dimension_routing --test phase1_validation --run-ignored all
  → 20 tests run: 20 passed, 0 skipped
86 + 20 = 106 = the pre-fix total, with 0 failures
```

## 4. Follow-ups register

| ID | Action | Depends on | Acceptance |
|----|--------|-----------|------------|
| ACT-399 | Merge #1179 (publish guard `inputs.dry-run != 'true'`, `Verify publish landed`, membership-shaped version check, gate-contract + publish-fixture sensors) | terminal-green CI incl. Codacy | merged to `main`; `./scripts/validate-gate-contract.sh --ci-parity` PASS |
| ACT-400 | Merge #1177 (readiness gate counts the **latest** run per check name) | after ACT-399 | merged; the stale `Check for unreleased changes` failures on #1179 no longer shadow its green re-run |
| ACT-401 | Merge #1175 (guardrail docs) then #1180 (nightly Turso) then this plans PR, serially | after ACT-400 | each merged via `./scripts/merge-pr.sh <n> --execute` only |
| ACT-402 | **0.1.46 bump PR** — workspace version, `Cargo.lock`, version docs, `## [Unreleased]` → `## [0.1.46]`; it is the vehicle that actually publishes | after ACT-401 | `do-memory-core`, `-storage-redb`, `-storage-turso`, `-mcp` all answer `index("0.1.46") != null` on crates.io **and** `Verify publish landed` green in every job |
| ACT-403 | Next nightly run green (scheduled + `workflow_dispatch`) | after #1180 merges | `gh run list --workflow nightly-tests.yml --limit 1` → `success`; the two feature-gated suites reported in the dedicated step, 20/20 |
| ACT-404 | `libsql 0.9.30 → 0.10.0` upgrade PR (both lockfiles) to clear 8 of the 12 Dependabot alerts | none | alerts 0; `cargo audit` clean; record `lru` as **upstream-blocked** by `augurs-changepoint`'s `rv 0.16` pin rather than pretending it is fixable |
| ACT-405 | Decide whether scheduled/tag-triggered nightly runs gate releases (`do_ci_check` `.event` filtering) | ACT-403 | written decision + ADR. **Hardening only** — never weaken the gate at ship time; the shipped answer for this release was "ship inside the window" with the red nightly explained above |
| ACT-406 | Schema-vs-SQL sensor: a new `validate-schema-sql.sh` (to live beside `validate-gate-contract.sh` in the scripts dir) compares columns referenced in hand-written SQL against the DDL constants, wired into `skill-evals.yml` | after #1180 merges | RC4 class of bug fails locally, not in nightly |
| ACT-407 | LESSON-055 + AGENTS.md trap bullet: feature-gated suites need `required-features` (not `cfg_attr(..., ignore)`) because nightly forces ignored tests; asserts must print the `Err`; a green job whose only real step is `conclusion=skipped` is not evidence | after #1175 merges (it owns both files) | entries land on a follow-up docs branch |
| ACT-408 | Close #1165 (v0.1.45 drift issue) with the release evidence, and re-check #1068/#1089/#1076/#1109 queue positions | ACT-402 | issue list matches the register in `STATUS/GAP_ANALYSIS_LATEST.md` |

## 5. Standing constraints this closeout was executed under

- **Every CI check must pass, including Codacy — nothing skipped, waived, bypassed or papered over.** No
  `--admin`, `--force`, `--no-verify`, no run deletions, no soft-pass on a required check.
- Releases only via the `release-guard` skill + `./scripts/release-manager.sh ship --execute`; **never**
  `gh release create`.
- Merges only via `./scripts/merge-pr.sh <PR> --execute`.
- Known pre-existing local-only failure, reproduced on untouched `main` and therefore not attributable to #1180:
  `cargo clippy --all-targets` reports unfulfilled `#![expect(...)]` in `test-utils/src/lib.rs` under the local
  floating stable. Tracked under ACT-404's toolchain hygiene, not silently "fixed" by suppressing.

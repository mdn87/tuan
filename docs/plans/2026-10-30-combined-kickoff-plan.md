# Tuan Kickoff Plan

Date: 2026-10-30. Status: proposed; no implementation or experiment is authorized by this document. Owner: Matt.

## What Tuan Is

Tuan is a standalone Lugos project for agent-work measurement, context-impact estimation, and usage forecasting. It preserves a trustworthy history of what was requested, what was expected, what context was supplied, what actually happened, and what the complete workflow consumed—then uses that history to improve forecasts and evaluate context strategies across agents, models, and repositories.

The name is **Tuán mac Cairell**, who preserves and recounts history across successive transformations. The package and submodule are both `tuan`.

### Ecosystem Ownership & Boundaries

| Component | Responsibility |
| --- | --- |
| **Tuan** | Normalize operational evidence; retain append-only history; build rebuildable analytical views; issue forecasts, findings, and reports; evaluate their usefulness and cost. |
| **Sulis** | Own durable memory content, capture, and recall. Supply versioned metadata and identifiers; Tuan never writes memories or duplicates their contents. |
| **Torc** | Own continuity, context construction, and context projection; serve as the source of prior efficiency research. Supply context manifests and policy/version metadata. |
| **Autowork & Harnesses** | Own task admission, scheduling, execution, routing, and any later approved intervention. Supply run lineage, checkpoints, and outcome evidence. |
| **Source Telemetry & Audit** | Remain authoritative for raw provider counters and closure receipts (`lugos-audit`, which attests repository hygiene at closure, not task behavior or acceptance). Tuan preserves provenance and reconciles definitions across sources. |

Tuan is not a second memory store and does not orchestrate execution. Its initial interfaces are local library functions, versioned schemas, and a CLI (`tuan`). Automatic memory edits, context tuning, agent routing, dashboards, and long-running background services are out of scope for the initial design and remain disabled until Milestone 5 defines their safeguards.

---

## Inputs & Prior Evidence

- `docs/research/context-impact-management.md` (renamed from root `context-impact-mamagement.md` in M0): the shaping conversation and revised standalone kickoff prompt.
- `docs/research/predictive-analytics.md` (moved from root in M0): predictor and observer architecture, checkpoints, evaluation design, and build-first progression.
- **Torc efficiency research on `origin/master` at `84c0ec8` (2026-10-22)**: The local `torc` submodule checkout is currently 16 commits behind at `26950f6` and does not yet show these files in the working tree; inspect them from `origin/master` or fast-forward via `scripts/git_sync.py`:
  - `docs/plans/2026-10-21-measured-efficiency-plan.md`
  - `docs/plans/2026-10-21-efficiency-execution.md`
  - `experiments/session-continuation-001/diagnostic-results.md`
- **Frozen historical finding to preserve unchanged**: In the one diagnostic workload where both portable approaches passed, Torc recorded `834,771` input tokens against `578,622` for the plain handoff (`44.27%` more). Verified on 2026-10-30 against `experiments/session-continuation-001/diagnostic-results.md` at `torc@84c0ec8` (workload `capsule-context`: `torc_capsule` 834,771 vs `compiled_prompt` 578,622 input tokens). Retain it as a result for that specific comparison (importable as a labeled historical dataset per D5), not a new Tuan result or a verdict on every context strategy or workload. Do not rewrite the old study during kickoff.

---

## Core Questions & Ground Rules

Tuan keeps three questions strictly separate:

| Capability | Question it answers | Key distinction |
| --- | --- | --- |
| **1. Observation** | What context, usage, active work, elapsed time, and outcomes occurred? | Direct accounting with provenance; missing is `unknown`, never `0`. |
| **2. Forecasting** | Given only the information available at a checkpoint, what total or remaining usage should we expect? | `Forecast − actual` is forecast error, not context savings. |
| **3. Attribution** | Compared with a specified baseline, what difference can reasonably be attributed to a context strategy or observer intervention? | Smaller initial context is not automatically lower total effort; cheap failure is not efficiency. |

### Reporting Rules

- Lead with the strongest supported measurement or estimate. Label every quantity **observed**, **estimated**, **experimental**, or **unknown**. Name the comparison baseline, accounting scope, and reporting period.
- State each material limitation once, then name the smallest next measurement that would reduce it. Never substitute a blanket "cannot prove savings" disclaimer for the quantified evidence already in hand.

### Evidence, Privacy, and Storage Rules

- **Storage architecture**:
  - **SQLite** is the canonical ledger: one single-writer database per host, authoritative for every record produced on that host (events, forecasts, corrections, allocation entries, deletion receipts), stored outside the git checkout and outside live file-synchronization folders; never sync a live SQLite file across hosts. Analytical projections are disposable tables rebuilt from ledger records under a versioned transformation.
  - **JSONL** is the complete, versioned, append-only export of a host ledger: the transport collected to `lugos-host` (which imports it idempotently into its own ledger), the backup representation, and the sanitized fixture format. The complete export set is the recovery set for an empty database.
  - **Shared database / collector service** is deferred until measured multi-host concurrency demands it.
- **Append-only and idempotent**: Re-importing the same event is idempotent. A duplicate identifier with differing content is an explicit conflict, never a silent overwrite. Corrections append a superseding record while keeping original forecasts and observations inspectable. Unknown schema versions or malformed records fail visibly or enter a sanitized quarantine.
- **Redaction before persistence**: Redact and allowlist fields before any persistence, including quarantine and rejection logs. Persist metadata and access-controlled references by default; never persist secrets, credentials, hidden reasoning, or unrestricted prompt, response, or memory contents. Payload capture requires an explicit purpose and opt-in retention class (off by default).
- **Retention and deletion**: Retain operational records for at least **90 days** by default (configurable to 180+ days per D4). Set a storage quota and warn before it threatens that window; never silently discard evidence to satisfy a quota. Retention applies to local exports and backups as well as the database, and reports state which evidence has expired and which replay claims remain possible. Append-only semantics prevent silent history rewriting; they do not override authorized privacy deletion, which must purge matching records, projections, exports, and managed backups across Tuan-owned storage classes while leaving only a non-sensitive deletion receipt so re-imports cannot resurrect purged data; producer-owned originals outside Tuan's authority are reported as remaining, never assumed purged. Compact evidence records may belong to a longer retention class than operational records (decided before the first M1 import); authorized privacy deletion applies across every class, including superseding scores, projections, and other derived records, and a rebuild or later maintenance run after deletion resurrects nothing.
- **Strict temporal cutoffs**: Every event records both *event time* and *ingestion time* in UTC alongside source clock quality and sequence numbers. Preserve known parent-child relationships; never invent them from proximity in time, and use sequence and lineage evidence where clocks disagree. A source timestamp earlier than a prediction is insufficient if the record had not been received by the forecasting ledger when the prediction ran. Late arrivals update later reports but never enter an earlier checkpoint's feature snapshot. Retrospective replays must be explicitly labeled; never manufacture a prospective forecast by backdating one.

### Accounting Rules

- **Provider-specific token categories**: Store input, output, cache-read, and cache-write counts separately with provider definitions. Cache counts are often subsets of reported input totals—never sum counters blindly.
- **Single-charge reconciliation**: Reconcile parent aggregates against an authoritative call-level source so cumulative-counter snapshots, retries, and parent/child calls never charge the same work twice. Counter resets are explicit discontinuities, not negative consumption.
- **Whole-workflow scope**: Include failed and abandoned attempts, retries, child calls, context preparation, retrieval, summarization, maintenance, and observer/reporting overhead. Report coding execution and observer overhead separately, plus an explicit whole-workflow total.
- **Reusable preparation allocation**: Track shared preparation in an allocation ledger (direct cost, allocated shares per task, allocation rule/cohort, and unallocated overhead). Never charge both full shared preparation and its allocated shares in the same total.
- **Distinct units**: Keep tokens, dated-rate monetary estimates, actual billing, subscription allowances, wall-clock elapsed time, active work, identifiable wait states (user, provider queue, build), and human effort distinct. Overlapping child durations cannot be summed into wall time; unobserved waiting or human effort stays unknown. Do not treat token counts across different model/tokenizer families as interchangeable effort units.
- **Quality pairing**: Track completion and quality alongside usage. Keep failed, abandoned, and in-progress runs in the history without treating unfinished cost-so-far as a cheap completed task.
- **Estimate and score lineage**: Keep four record kinds distinct and never substitute one for another: **original forecast** (what a predictor emitted at a checkpoint under its information cutoff), **recorded actual** (the labeled outcome with its label version), **recalculated score** (a scoring-rule version applied to preserved forecasts and actuals under the Score contract; supersedes earlier scores without deleting them), and **revised estimate** (a newer predictor, training snapshot, or predictor-input transformation replayed against a historical checkpoint; always `retrospective_replay` with a leakage status). Evaluation-only changes rescore unchanged predictions; a change that alters predictor inputs requires new predictions, which are revised estimates. A revised estimate is never scored as prospective forecast accuracy, whatever its leakage status, and never stands in for a missing actual; comparing two estimates measures their agreement only. A missing actual stays `unknown`.

---

## Minimum Record Contracts

| Record | Required information |
| --- | --- |
| **Event envelope** | Immutable source content: event/schema/source versions; stable event ID and source deduplication key; UTC event time and source sequence; event type; task/run/attempt/session/agent IDs; parent link; host (`lugos-link` slug) + process start time + PID; provenance reference; redaction status. Separate ledger receipt: collector identity, UTC ingestion time, and a monotonic ingestion position in the receiving ledger. Duplicate-content comparison covers the source content only; the receiving ledger's receipt governs checkpoint cutoffs. |
| **Task / run** | Request reference and version; task family and repository; acceptance criteria and revision; provider/model/settings; lifecycle state; parent-child lineage; context policy/strategy version and other configuration identifiers recorded as *observed in effect* for the run (declared-versus-observed provenance flagged; mid-run changes ordered by event sequence), never inferred from a decision's acceptance date; applicable decision references (namespaced decision ID, immutable revision, lifecycle status at run start, applicability source) when a producer or operator supplies them. Missing links are explicit; `unknown` is a valid value. |
| **Usage observation** | Provider call ID; accounting scope and role (`coding`, `preparation`, `observer`, `reporting`); reported metric categories/units; delta vs. cumulative semantics; measurement provenance/quality; dated rate or billing reference if available. |
| **Checkpoint** | Immutable checkpoint ID; available-through ingestion boundary and source positions; task/spec/plan revisions; git commit SHA plus patch hash for uncommitted work; context references; test results and usage known so far; feature snapshot and version. |
| **Forecast** | Target (`total_input_tokens`, `total_output_tokens`, `remaining_input_tokens`, etc.), units, and a versioned target signature (included roles, lineage scope, allocation-policy version, terminal event, label-availability rule); checkpoint ID and information cutoff; prediction timestamp; estimator and training-snapshot versions; point estimate and empirical/calibrated range (or `insufficient_history`); assumptions; inference cost and latency; forecast kind (`prospective` or `retrospective_replay`) and, for replays, a leakage status: `cutoff_verified` (training-snapshot and feature-snapshot bounds demonstrably at or before the checkpoint's information cutoff), `contaminated` (training or features known to include the task's outcome or later records), or `unknown` (provenance unavailable; never upgraded without evidence). Revisions link to prior forecasts without overwriting. |
| **Outcome** | Status (`completed`, `failed`, `abandoned`, `in_progress`); acceptance evidence (test output, verification records; a `lugos-audit` receipt attests hygiene, not acceptance) and assessed revision; final or so-far cost with its label version; quality assessment; outcome-known timestamp; linked rework updates, keeping later operational spending apart from corrections to spending inside the closed target and preserving every label revision. |
| **Score** *(derived, M2)* | Score ID derived deterministically from its binding so repeated evaluation is idempotent; bound forecast ID (or an immutable aggregate membership list), outcome and label revision, target and metric, scoring-rule implementation and configuration version, weighting-policy ID and version when applicable, evaluation-as-of time, and the superseded score ID if any. A current view selects the latest score per binding within one scoring-rule and policy series; incompatible metric or policy series are never merged, and the unweighted series is always retained. |
| **Finding** *(M4)* | Claim; affected requirement or file/line location; evidence references; inspected revision; expected consequence; suggested verification; observer version and cost; advisory disposition (`surfaced`, `deduplicated`, `stale_revision`, `dismissed`). |
| **Intervention** *(M5)* | Finding or policy reference; advice shown and timestamp; actor/decision; action taken; policy revision; verified result; rollback reference. |

### Planned Interfaces

- **Adapter**: source records → validated normalized events with provenance and import coverage.
- **Projector**: retained events + transformation version → reproducible analytical views.
- **Predictor**: checkpoint + explicitly bounded training snapshot → independent forecast or explained unavailable result.
- **Reporter**: versioned views + report scope → machine-readable JSON and readable Markdown.
- **Observer** *(M4)*: checkpoint + bounded evidence → revision-specific findings, with no authority to modify the task.

Compare predictors on the same target, checkpoint, and information cutoff, and save each original output before exposing any peer output. Physical parallel execution is optional; independence and comparable inputs are the requirements.

---

## Repository Layout & Cross-Platform Lanes

```text
tuan/
  README.md                         concept, boundaries, POSIX & PowerShell quickstarts
  AGENTS.md                         concise project working rules (with CLAUDE.md pointer)
  LICENSE                           matching sibling Lugos projects
  pyproject.toml                    Python package, runtime floor, pytest config
  .gitignore / .gitattributes       ignore .lugos-audit/, .lugos-wip/, operational data
  docs/
    inventory.md                    verified reuse / extend / build inventory + stale-doc list
    concept.md                      plain-English concept, ideal vs. practical design
    contracts.md                    event, checkpoint, accounting, forecast, finding schemas
    evaluation.md                   cohorts, chronological splits, metrics, promotion rules
    divergence-register.md          compromises and revisit triggers
    decisions/                      recorded operator decisions (D1–D6)
    plans/                          kickoff and milestone implementation plans
    research/                       context-impact-management.md, predictive-analytics.md
    studies/                        attribution study protocols (M5)
  schemas/                          JSON Schema / contract definitions
  src/tuan/
    cli.py                          import, replay, forecast, report entrypoints
    contracts.py                    versioned record models and validation
    adapters/                       source adapters (starting with one verified workflow)
    store.py                        SQLite event ledger, JSONL import/export, projections
    accounting.py                   provider normalization and allocation ledger
    checkpoints.py                  bounded evidence snapshots and cutoff enforcement
    forecasting.py                  historical baseline and regularized regression challenger
    evaluation.py                   chronological splits, interval scoring, recent-error tracking
    reporting.py                    JSON and Markdown report generation
  tests/
    fixtures/                       sanitized real exports and labeled synthetic edge cases
```

- **Keep modules cohesive**: The files under `src/tuan/` define subsystem boundaries, not a mandate for premature abstraction. Observer and advisory modules are added only in M4.
- **Platform matrix**: Target macOS (`applemac`), Windows (`4070pc`, `3060pc`), and Linux (`lugos-host`). Use `applemac` as the first local verification host; verify Linux and Windows separately before claiming support. Share Python logic and schemas; isolate OS divergence to virtual-environment activation, paths, file permissions, and SQLite locking. Document POSIX and PowerShell quickstart lanes, and test paths with spaces and interrupted imports on each supported platform.
- **CI runner policy**: Before adding CI, verify repository visibility (D1) and runner availability. Private-repository jobs use `[self-hosted, lugos-host]`; if no such runner is registered for `tuan`, ask Matt before adding CI, keep local checks available, and never substitute a hosted runner. Public repositories use hosted runners. Any hosted Windows/macOS job runs only on manual `workflow_dispatch` or a schedule of at most once a week, never on push, pull request, or tags.

---

## Phased Milestones

**First release = M0 → M1 → M2.** Milestones M3–M5 build on verified M2 history.

### M0: Repo Seed, Discovery, and Contracts

**Already completed**: repository created, seed docs pushed, `tuan` registered as a Lugos submodule (`mdn87/lugos#682`).

1. **Repo hygiene**: Add `README.md`, `AGENTS.md` (with `CLAUDE.md` pointer), `LICENSE`, `pyproject.toml`, `.gitignore` (`.lugos-audit/`, `.lugos-wip/`, local databases/exports), and `.gitattributes`. Move the two root seed docs into `docs/research/` and fix the filename typo (`context-impact-management.md`).
2. **Evidence-backed inventory (`docs/inventory.md`)**: Inspect actual source and deployment state (separating *planned*, *implemented*, and *deployed*) across:
   - **Torc** (`origin/master` at `84c0ec8`): continuation accounting, context projection, `experiments/session-continuation-001/`, `docs/attempt-accounting.md`, `docs/evaluation-plan.md`.
   - **Sulis**: versioned metadata and identifiers exposed during memory capture and recall.
   - **Autowork**: campaign request envelopes, run identities, and `logs/`.
   - **Toolset & Harnesses**: `lugos-audit` receipts, `agent-journal` and `verification-evidence` skills, Claude Code and Codex session transcripts/usage logs on each host, and `lugos-link` host identity slugs.
   - **Decision and configuration identity**: which producer, if any, exposes decision references and effective configuration identity on each host platform, mapped per field as native, operator-supplied, derived, or unavailable, with a representative fixture per class; M0 also fixes the decision-ID namespace, revision representation, applicability sources, historical status evidence, configuration identity scope, declared-versus-observed provenance, and the ordering of mid-run changes in `docs/contracts.md`. Where no producer exposes a field, record the gap under the selection rule rather than inventing values; population feasibility is unverified until this mapping exists, and `unknown` remains a valid M1 outcome.
   - **Selection rule**: Select the first workflow (D2) by evidence quality: stable lineage, provider usage, context references, and explicit completion checks. Prefer a file export or an existing event interface over installing instrumentation. If an essential field is absent, document the gap and add the smallest producer-side extension as a separately scoped change; do not alter producer schemas or original studies merely to make the fixture fit.
3. **Stale documentation & study gate classification**:
   - List stale or conflicting docs that cause agents to repeat inaccurate disclaimers (specifically the Torc `README.md` offline-milestone summary vs. the `2026-10-21` measured efficiency plan) and specify the exact fix needed.
   - Classify each gate from the frozen Torc study as *study-specific* or *adopted by Tuan*, with the rationale.
4. **Core design docs & schemas**: Draft `docs/concept.md`, `docs/contracts.md`, `docs/evaluation.md`, `docs/divergence-register.md`, and `schemas/`, and record decisions **D1–D6** in `docs/decisions/` as numbered `NNNN-slug.md` records (Torc's style; IDs namespaced `tuan:NNNN`) with Date, Context, Decision, Consequences, Alternatives Considered, and four evaluable fields: **Expected benefit** (observable; names the Tuan record or view that would show it and the earliest point it could be observed), **Accepted downside**, **Measurement** (what is compared against which baseline, noting that task mix, model, and caching changes can explain a difference), and **Reconsideration trigger** (a condition on retained evidence). Decision content is immutable once accepted and its revision identity is the accepting commit; lifecycle (`proposed`, `accepted`, `superseded by NNNN`) is recorded as dated entries in `docs/decisions/lifecycle.md` and successor links, never by editing an accepted record. A decision applies to a run or inspected revision when it was accepted and not yet superseded at that time; a `proposed` record is an option and never binds a run.

**Exit criteria**: `lugos-audit quick` passes, package imports cleanly in pytest, `docs/inventory.md` verifies every reuse/extend/build row with exact paths/commits, one sanitized real-workflow sample fixture is captured (covering request, context, call usage, lineage, and outcome, with missing fields and ownership identified; a synthetic fixture supports development but does not satisfy real-workflow acceptance), decisions D1–D6 are recorded in `docs/decisions/` (resolved from evidence where possible; Matt reviews the design package and is asked where source access or the intended workflow stays ambiguous), and the parent submodule pointer is bumped via `scripts/git_sync.py bump`.

### M1: Replayable History & Trustworthy Accounting

**Outcome**: Ingest exported records from the first selected workflow (D2) into the SQLite ledger and deterministically rebuild analytical views.

1. Implement `contracts.py`, the first workflow adapter in `adapters/`, `store.py` (SQLite ledger + JSONL import/export + versioned projections), `accounting.py` (provider token normalization + preparation allocation ledger), and `checkpoints.py`.
2. Implement pre-persistence redaction, quarantine for malformed/unknown-schema records, and the authorized privacy deletion receipt flow.
3. Import the frozen Torc study dataset (if approved in D5) as an explicitly labeled historical cohort.

**Exit criteria**: Pytest suite covers idempotent re-import, conflicting duplicate IDs, cumulative vs. delta counters, cache-subset accounting, parent/child call overlap, failed/abandoned attempts, late arrivals, clock skew, unknown schemas, pre-persistence redaction, and deletion persistence across replay, plus: a record collected on one host before a forecast but received by the forecasting ledger afterward stays excluded from that forecast and raises no duplicate conflict on repeated import or clean rebuild; a fixture where one preparation operation serves several tasks and leaves an unallocated remainder verifies that allocated shares plus the remainder equal the direct cost, that task and cohort reports (including one restricted to a single benefiting task) use their declared scope without charging the direct cost again, and that replay preserves the allocation-policy version. Rebuild into an empty SQLite database from the complete export set matches normalized totals exactly, and one real completed task reconciles against its raw source log with any coverage gaps explicitly labeled. Before the first import, storage is classified as producer-owned originals, Tuan-owned ledgers and exports, host spools, or managed backups, with retention and quota behavior set per managed class; migrations and rebuilds never modify producer-owned originals, and deletion followed by rebuild or re-import from an older managed export or a previously offline host does not resurrect purged records. Collection/replay time and storage size are benchmarked rather than assumed negligible, including the long-lived compact evidence class if adopted. A retention fixture with controlled timestamps expires operational records while retaining compact evidence records, then verifies which summaries remain valid, which replay claims become unavailable, and that reports say so; authorized deletion is exercised across originals, revisions, superseding scores, projections, and other derived records, and a rebuild or later maintenance run after deletion resurrects nothing.

### M2: Competing Shadow Forecasts & First Report (First Release Stopping Point)

**Outcome**: Complete the smallest useful end-to-end loop without background services or autonomous intervention.

1. Implement two local statistical predictors in `forecasting.py` running independently in shadow mode against identical checkpoint cutoffs. Forecast total reported input and output tokens as separate targets first; add remaining-usage forecasts at later checkpoints once total-usage accounting is reliable, defined relative to usage known at that checkpoint, with late accounting corrections governed by the target signature's label rule. Defer elapsed-time and failure-risk models until their labels are dependable.
   - **Champion (Historical Baseline)**: median and empirical range from compatible prior tasks (emitting explicit `insufficient_history` or labeled broader-cohort fallback on cold start; synthetic data can prove the interface works, never forecasting quality).
   - **Challenger (Regularized Regression)**: lightweight local regression over a few explicit task features (task family, model route, starting context size, repository characteristics—keeping difficulty multi-dimensional rather than a single invented score). The challenger stays unavailable while training or validation support is inadequate.
2. Implement `evaluation.py` (grouped chronological splits keeping task lineages together, frozen training snapshots, point error and signed bias, underestimation rate/magnitude, empirical interval coverage and width, inference cost, performance by cohort, each score recorded under the Score contract with the label version and scoring-rule version it used) and `reporting.py` (JSON + Markdown). Label empirical ranges uncalibrated until held-out coverage supports a stated level. Define promotion criteria (required improvement, tolerated underestimation, interval coverage, quality/coverage constraints, overhead budget) before scoring a candidate; until they are met, display the baseline and keep the challenger in shadow mode.
3. Expose `tuan import`, `tuan replay`, `tuan forecast`, and `tuan report` in `cli.py`.

**Exit criteria**:
- **Retrospective reconstruction**: From a clean checkout, follow `README.md` to reconstruct one completed coding task across its checkpoints—showing what each predictor knew at the cutoff, what it predicted, what actually happened, and what observation/forecasting cost. Re-import/replay produces equivalent results; replays are labeled retrospective; cold start, delayed outcomes, aborted tasks, a changed context/code revision, a late usage correction, a report rerun, and post-completion rework are exercised, and adversarial cutoff tests leave forecast inputs and outputs unchanged when future records are added (a prior task whose outcome becomes known after the cutoff, a late-arriving record with an earlier source timestamp, a referenced artifact changed after checkpoint capture). A label correction, a scoring-rule change, and a repeated evaluation run produce superseding recalculated scores that neither duplicate contributions nor erase earlier scores or the unweighted reference, leaving original forecasts and actuals inspectable; a predictor-input transformation fitted on later outcomes never silently regenerates predictions; replays with a newer predictor are reported as revised estimates with leakage status, exercised with a clean snapshot, outcome-contaminated training, later-record features, and unavailable provenance, and every revised estimate stays excluded from prospective accuracy whatever its status.
- **Prospective verification**: Observe one subsequent real task through an existing export/checkpoint interface where forecasts are persisted *before* the outcome exists—evidence of forecast-before-outcome ordering; leakage-free enforcement rests on the cutoff tests above. Where history is insufficient, preserve the explicit unavailable result rather than backfilling. Run no additional coding workload solely to manufacture results without separate authorization. Report which parts are replay-tested and which have prospective evidence.
- Passing requires no savings claim and no champion replacement; `insufficient_history`, negative, and incomplete results are valid outputs when clearly reported.

### M3: Live Checkpoint Capture & Deterministic Rule Signals

**Outcome**: Accumulate routine operational history for D6 data sufficiency with bounded, measured collection overhead.

1. Wire lightweight checkpoint emission into the D2 workflow (using existing lifecycle hooks or file exports, not a persistent daemon) at six meaningful events:
   - `plan_ready`, `patch_produced`, `repeated_test_failure`, `scope_change`, `compaction_or_handoff`, `completion_claimed`.
2. Add deterministic, zero-LLM **loop and risk detector rules**:
   - Repeated reads of unchanged files after a context reset;
   - Identical test failure repeating without a changed assumption;
   - Reverting and recreating substantially similar patches;
   - Diff growth while acceptance criteria remain unresolved;
   - Growing effort spent recovering previously supplied information;
   - Completion claimed while an explicit requirement has no verification evidence.
   *(Each rule emits an inspection signal, never an automatic failure verdict.)*
3. Add **`tuan refresh`**, an idempotent maintenance verb. It derives pending work from durable **evaluation-coverage records** (which forecasts and lineages have been scored under which scoring-rule and policy versions), kept independently of projection progress so scoring missed after a projection advance or an interruption is rediscovered: new events or newly known outcomes → project and score affected lineages; a changed evaluation-only scoring rule → superseding recalculated scores; a changed predictor-input transformation or predictor version → retrospective revised estimates only when explicitly requested; a crossed recency-window boundary → refreshed weighted scores; nothing relevant → a recorded no-op. Score identities are deterministic so repeats are retry-safe, and coverage progress commits only after a unit's scores are durably written. Receipts (what changed, what was recomputed, versions, duration) are operational records written outside the evidence ledger, and receipt-only changes never count as pending work. It never retrains or promotes a predictor and never modifies ledger originals; running it from cron on `lugos-host` is an operations decision taken after its cost is measured here, and it remains a periodic batch, not a service.
4. Add an opt-in, versioned **recency-weighting policy** for scorecards, off until a recorded decision selects it; the unweighted score stays the reference and promotion criteria stay unweighted. The policy and each weighted score's evaluation identity fix the window size and basis (outcome-known time, or completed-lineage count with deterministic ordering), the evaluation-as-of time, the eligible cohort and lineage population, the contribution unit, and the label-selection rule; late outcomes and corrected completion times create a new as-of evaluation rather than rewriting an earlier one, so a current reassessment stays distinguishable from a historical-as-of result. Weights are group-normalized so a stated split (such as 65/35) holds regardless of group sizes, which is disclosed as intended; below a declared minimum recent support the weighted score is suppressed as `insufficient_recent_support`, and an empty group falls back to the unweighted score with an explicit note.

**Exit criteria**: Source-to-ledger reconciliation survives process restarts and reconnects without duplicate charges; collection errors fail open for the coding agent while surfacing visibly in Tuan; missed events, ingest latency, storage growth, and overhead are measured against a declared budget; configured retention is verified with controlled timestamps; prospective evaluation cohorts are established before predictors are judged; each supported checkpoint event has a trigger test; each supported rule declares its required metadata, predicate or threshold, and behavior when inputs are unavailable, with one positive and one nearby negative fixture, and rules whose inputs are unavailable are deferred rather than reading missing evidence as no risk. Releasing does not require waiting 90 days. Refresh runtime, storage growth, and no-op cost are measured against the declared budget; tests cover already-projected but unscored events, interruption between projection and scoring, late outcome corrections, repeated execution, and a no-op that leaves the evidence ledger and projections byte-identical while writing only its operational receipt. Weighted-score tests cover a window-boundary crossing, a late correction, multiple forecasts in one lineage, the minimum-support and empty-group boundaries, and rebuilding the same as-of evaluation after later events arrive.

### M4: Bounded Semantic Observer & Advisory Stage

**Outcome**: Measure whether one narrow LLM reviewer adds confirmed value beyond deterministic rules.

1. Add one semantic observer with a single narrow job—detecting **unsupported completion claims or requirement drift**—invoked only at selected completion/patch checkpoints under a strict cost cap.
2. Enforce observer isolation: record the observer's raw finding before exposing any peer output; bind every finding to the exact code revision inspected; use an isolated git worktree/checkout if verification runs tests.
3. Add the single **advisory stage** that deduplicates findings, checks whether the inspected revision is still current (requiring revalidation if relevant code changed), and decides what to surface (in shadow mode first).
4. Score the observer on **confirmed findings, missed problems on adjudicated samples, false alarms, useful warning lead time, interruptions, and token/latency cost** (never agreement rate). Preserve findings that prompted a fix as confirmed true positives rather than relabeling them false alarms after the defect disappears.
5. Define, without scheduling, two decision-aware finding types for later specialists under the "measured gap" rule: **`decision_conflict`** (a patch or run contradicts a decision accepted and not superseded at the inspected revision's time; cites decision ID and revision, affected location, and expected consequence; deterministic where the decision carries a machine-checkable constraint, otherwise semantic) and **`revisit_proposal`** (a divergence-register or decision reconsideration trigger appears satisfied by retained evidence; cites the original decision or compromise, the evidence that its constraint changed, affected components, and estimated effort and benefit). Both are advisory and bound to the inspected revision like every Finding.

**Exit criteria**: The observer detects seeded unsupported claims, leaves a supported claim unflagged, preserves a stale result against its original revision, and requires revalidation when relevant code changes (functional checks); rules and the semantic observer are then scored on the same bounded checkpoint sample with independently adjudicated positive and negative cases, reporting incremental confirmed findings, misses, warning timing, duplicate findings, and added cost with explicit denominators—zero or negative incremental value is an honest result. Requires reliable M2 records and an explicit model/cost choice before any paid call. Add other specialists only for a measured gap; the execution system remains responsible for any response to advice.

### M5: Attribution Studies, Ensembles, and Controlled Improvement Loop

**Outcome**: Answer specific context-impact questions at stated confidence levels and govern any policy recommendations.

1. **Attribution progression**: Move from descriptive reporting → clearly labeled observational comparisons (with declared matching/adjustment variables and confounders) → human-approved matched or randomized task-lineage trials under `docs/studies/`.
2. **Separable causal questions**: Keep four questions distinct in every study protocol:
   - Does the context treatment itself reduce total effort or improve quality?
   - Do observer warnings/interventions help?
   - Does the combined workflow justify its total preparation + coding + observer overhead?
   - Did the expected benefit stated in a decision record materialize, measured as that record's Measurement field specifies, with its causality caveat?
3. **Ensembles**: Evaluate predictor ensembles only after M2–M3 history measures individual predictor error correlation across cohorts.
4. **Governed improvement ("self-healing" contract)**: Progress from reports → recommendations → human-approved context-policy experiments. Any future automatic policy adjustment requires a written contract specifying: what signal detects a problem, the bounded parameter allowed to change, versioned policy IDs, quality guardrails, an independent evaluation holdout that the policy cannot manipulate, full audit trail, automatic rollback, and hard stop conditions. Optimizing usage alone, or letting the evaluator rewrite its own evidence, does not satisfy that contract.

**Exit criteria**: The study protocol, cost-allocation rule, quality criteria, and stopping/reporting rules are written before the comparison is examined; reported group totals, effect estimates, uncertainty, exclusions/missingness, and quality outcomes are reproducible from retained evidence; preparation, retry, observer, and maintenance costs are included; a measured disadvantage is reported as readily as a benefit, with the smallest next measurement recorded. No automatic context or memory change follows from a favorable report.

---

## Illustrative Report Contract

Every Tuan report leads with the strongest supported observation, distinguishes **observed**, **estimated**, **experimental**, and **unknown** values, states each limitation once, and names the smallest next measurement.

*(The numbers below are purely illustrative to define the report contract, not project measurements.)*

| Report Question | Illustrative Output |
| --- | --- |
| **What happened? (Observed)** | One accepted task consumed `60,000` reported input tokens and `8,000` output tokens across all included roles. Acceptance checks passed for the recorded revision; actual billing is `unknown` unless an authoritative billing source supplies it. |
| **What overhead was included? (Observed)** | Of the `60,000` input tokens, `5,000` were context preparation and `2,000` were Tuan observation/forecasting. These are subsets of the workflow total, not extra charges. |
| **What about cache and time? (Observed / Unknown)** | Provider reported `20,000` cached input tokens (subset of the `60,000` input total). Wall-clock elapsed time was `25m` (`20m` active work, `5m` recorded build/user wait). Unlogged human review effort is `unknown`. |
| **What did we expect? (Estimated)** | Pre-execution baseline forecast: `70,000` input tokens (uncalibrated empirical range `50,000–100,000`). Shadow regression challenger: `66,000` (`48,000–90,000`). |
| **What was the forecast error? (Observed)** | Baseline overpredicted input by `10,000` tokens (`+16.7%`); challenger overpredicted by `6,000` (`+10.0%`). Single-task error is recorded for cohort scoring and is not evidence of a model winner. |
| **What context-related difference was established? (Unknown)** | `unknown` for this single task—no matched baseline under an alternative context policy was run, and forecast error (`−10,000` vs. baseline expectation) does not measure context savings. |
| **What is the smallest next measurement?** | Keep this task in forecast scoring; collect comparable tasks in this repository cohort under the specified alternative context policy, matched against the baseline handoff policy, before estimating an observed group difference. Once comparison evidence exists, add the observed difference and, separately, any supported context-effect estimate with its assumptions and uncertainty. |

---

## Divergence Register (Ideal vs. Kickoff Practical Design)

| Ideal Capability | Kickoff Choice (M0–M2) | Revisit Trigger |
| --- | --- | --- |
| **Multi-source live telemetry bus** | Single verified workflow adapter via JSONL export/hooks into local SQLite | First workflow reconciles cleanly and a second source adds distinct coverage. |
| **Centralized multi-host database** | Per-host single-writer SQLite ledgers; complete JSONL exports collected to `lugos-host`; disposable projections | Measured multi-host query latency or concurrency exceeds file-collection workflow. |
| **Calibrated prediction intervals & ensembles** | Historical median/IQR baseline + 1 regularized regression challenger; explicit `insufficient_history` on cold start | Held-out chronological lineages reach D6 threshold and predictor error overlap is measured. |
| **Panel of specialist LLM observers** | Deterministic rules first (M3), then one narrow completion-claim reviewer (M4) | Scorecard proves net confirmed findings and lead time justify observer token cost. |
| **Causal context-savings proof** | Descriptive accounting first, then labeled observational cohorts, then opt-in trials (M5) | Stable task-lineage cohorts and verified context-policy metadata exist. |
| **Full prompt/response forensics** | Redacted metadata and content hashes/references only; payloads off by default | A specific failure class requires opt-in payload retention under approved access controls. |
| **Autonomous self-healing routing/context** | Read-only reports and human-approved experiments only | Bounded authority, independent holdout evaluation, and rollback guards are verified in M5. |
| **Replay after schema changes** | Versioned transformations and retained events | A schema migration must pass replay parity before adoption. |
| **Replay after deletion/expiry** | Honest partial coverage | Never reconstruct erased content; revise dependent claims instead. |
| **Adaptive recency-weighted estimators** | Unweighted baseline; opt-in, versioned weighted scorecards from M3 | The weighted/unweighted gap is material across cohorts over the D6 history. |

A divergence is resolved only by (a) changing the implementation to match the concept, (b) a recorded, deliberate concept change with its evidence and a superseding decision record, or (c) recording it as accepted with its revisit trigger. Editing the ideal concept to match what was built, without that record, is not a resolution. Revisit triggers are evaluated against retained evidence, manually at milestone boundaries until `revisit_proposal` findings exist (M4); a satisfied trigger produces a review proposal, never an automatic change.

---

## Decisions Needed from Matt (D1–D6)

| ID | Decision | Proposed Default (if approved as-is) |
| --- | --- | --- |
| **D1** | **Repository visibility** (`mdn87/tuan` is currently public) | Keep **public** for code and schemas; strictly gitignore all operational data, exports, and SQLite files; use free hosted Linux CI. |
| **D2** | **First instrumented workflow** (`Claude Code` via hooks on `applemac`, `Autowork` campaigns on `4070pc`, or `Codex`) | **Claude Code on `applemac`** (where session transcripts and hooks already run locally). |
| **D3** | **Event store topology** (per-host files collected to `lugos-host` vs. `lugos-host` only) | **Per-host single-writer SQLite ledger** outside the repo, with complete versioned **JSONL** exports collected to `lugos-host` and rebuildable analytical projections (the authority model under Storage architecture). |
| **D4** | **Operational retention & payload opt-in default** | **180 days** operational retention (exceeding the 90-day floor); **payload capture off** (metadata and references only). |
| **D5** | **Import the frozen Torc `session-continuation-001` trial data** | **Yes**, imported in M1 with explicit provenance (`torc@84c0ec8`) and labeled as the frozen diagnostic study. |
| **D6** | **Data-sufficiency threshold for the first champion/challenger promotion check** | Set in `docs/evaluation.md` from the M0 inventory based on **usable independent task lineages, cohort representation, and the uncertainty around the promotion decision**, never an arbitrary count of days or tasks. If estimates stay too uncertain, retain the baseline and keep collecting. |

### Decisions Resolved at Their Point of Need

- **During M0**: exact first workflow/export, source owners, runtime floor, operator data location, how accepted completion is established, and the versioned forecast target signature. Resolve from evidence where possible; ask Matt only when source access or the intended workflow remains ambiguous.
- **Before the first M1 import**: storage ownership classes and the retention and quota behavior of each managed class, including whether to define a **long-lived compact evidence class** (original forecasts, recorded actuals and label revisions, recalculated scores, decision and configuration references, identities and versions; no payloads), proposed at three years, distinct from the 180-day operational class (D4) and any payload class, with its class assignment, expiry clock, and dependency behavior defined in the same decision.
- **Before operational collection (M3)**: quota size, retention above the 90-day default if needed, any payload opt-in, and whether to enable the recency-weighting scorecard policy. Metadata-only and unweighted are the defaults.
- **Before model promotion**: acceptable underestimation, empirical coverage, and overhead bounds. Until then, display the baseline and keep challengers in shadow mode.
- **Before semantic calls or intervention trials (M4–M5)**: the narrow scope, spending limit, and experiment conditions. No such execution is authorized by this plan.

---

## Key Risks & Mitigations

- **Double counting cumulative counters, cache subsets, or parent/child calls** → Per-provider accounting schemas and reconciliation tests in M1 before any predictor runs.
- **Hindsight or future leakage into forecasts** → Mandatory `ingestion_time` cutoff on every checkpoint and forecast; grouped chronological splits keeping task lineages together.
- **Stale docs causing agents to repeat "no token evidence"** → M0 stale-doc remediation list + mandatory reporting rules in `AGENTS.md`.
- **Observers mutating or slowing the active workspace** → Read-only observers by default; isolated git worktrees for any M4 verification check; hard cost/invocation budgets.
- **Process identity collisions across hosts or restarts** → Mandatory `(host_slug, process_start_time, pid)` tuple plus parent operation links in every event envelope.
- **Accidental commit of operational telemetry to a public repo** → Pre-persistence redaction, an external data directory chosen in M0 (outside the checkout and live sync folders), `.gitignore` guards, and M0–M2 fixture sanitization checks.

---

## Execution & Final Verification

Implement the first release directly in coherent batches, because storage, accounting, and forecasts share contracts. Independent adapter work becomes useful only after those contracts stabilize; give each adapter a disjoint write area and a shared conformance suite. Do not create one agent or reviewer per module.

At the end of M2, perform one consolidated self-review and run fresh contract, accounting, replay, and end-to-end checks. Inspect the final diff for unrelated edits, sensitive fixtures, accidental services, and unsupported capability claims. Document which platform checks passed and which remain pending. Broaden review only for a material unresolved integration or data-integrity risk.

Seed coverage: inventory and ownership → M0; replay, retention, and accounting → M1; immutable forecasts, competition, evaluation, and reports → M2; proactive history, rules, maintenance, and scoring policy → M3; semantic observers and advisory handling → M4; context attribution and controlled experiments → M5. The ideal concept and divergence register remain alongside the practical milestones.

**Recommended next action**: when implementation is requested, start M0 and carry through M2 to produce the first replayable real-workflow report. Do not begin a collector deployment, paid review, experiment campaign, or autonomous adjustment as part of repository kickoff.


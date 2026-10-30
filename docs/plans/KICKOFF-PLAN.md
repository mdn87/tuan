# Tuan repository kickoff plan

Date: 2026-10-30. Status: proposed implementation plan; no implementation or experiment is authorized by this document.

Source material: [context-impact-mamagement.md](context-impact-mamagement.md), especially its revised standalone-project prompt, and [predictive-analytics.md](predictive-analytics.md). The later standalone-project direction supersedes the earlier Torc-subproject proposal. Tuan is the working name, following this repository. This plan does not incorporate the document being written by Claude in parallel.

## Goal and first useful result

Tuan will preserve a trustworthy history of agent work: what was requested, what was predicted, what context was supplied, what happened, and what the complete workflow consumed. It will use that history to improve forecasts and evaluate changes to context strategies across agents, models, and repositories.

Keep three questions separate:

1. **Observation:** What usage, time, work, context, and outcomes were recorded?
2. **Forecasting:** Given the information available then, what total or remaining usage should we expect?
3. **Attribution:** Compared with a specified alternative, what difference can reasonably be attributed to a context intervention?

The first useful release is one workflow, one replayable history, a cheap baseline and one shadow challenger, and a readable report. It should let an operator reconstruct one completed task from request through acceptance, inspect each checkpoint's available evidence and predictions, and reconcile its reported usage with source records. A subsequent prospectively observed task must demonstrate that forecasts are saved before their outcomes exist.

Success does not require proving savings or choosing a winning model. Negative results, incomplete measurement, and a challenger that cannot yet be trained are valid outputs when clearly reported.

## Scope and ownership

| Component | Responsibility |
| --- | --- |
| Tuan | Normalize operational evidence; retain history; build analytical views; issue forecasts, findings, and reports; evaluate their usefulness. |
| Torc and the existing memory system | Own context construction, continuity, retrieval, and memory content. Supply versioned references and relevant metadata. |
| Autowork or the existing executor | Own task admission, scheduling, execution, routing, and any later approved intervention. Supply run lineage and outcomes. |
| Existing telemetry/accounting systems | Remain authoritative for their source measurements. Tuan preserves provenance and reconciles their definitions. |
| Existing UI/report consumers | Display Tuan outputs and collect operator decisions. A new dashboard is unnecessary for the first release. |

Tuan must not become a second memory store or executor. Its initial interfaces are local imports, library functions, and a CLI. No daemon, message broker, model-provider integration, dashboard, automatic memory edits, context tuning, or routing changes are needed to establish the first result.

The ideal system supports multiple evidence sources, independently evaluated observers, calibrated forecasts, and controlled context-policy experiments. The practical implementation starts with a local, single-writer data store and one adapter. Preserve a short divergence register as those choices evolve.

## Discovery before committing to integrations

The two seed documents establish intent, not the current implementation or deployment state of neighboring projects. Do not claim an integration exists until its source and a representative record have been inspected.

The first implementation task produces `docs/inventory.md` with rows containing capability, owning project, exact source path/revision, exported record or interface, deployment evidence, reuse/extend/build decision, and missing connection. Keep planned, implemented, and deployed states separate.

| Candidate | Evidence to inspect | Decision it must resolve |
| --- | --- | --- |
| Torc measurement work | `docs/plans/2026-10-21-measured-efficiency-plan.md`, `docs/plans/2026-10-21-efficiency-execution.md`, `experiments/session-continuation-001/diagnostic-results.md` in the Torc checkout | Reuse accounting and evidence formats where sound; preserve the original study and its gates. |
| Torc/current memory system | Context manifests, policy/version identifiers, retrieval and continuity records | Which metadata can describe supplied context without copying memory content? |
| Autowork/current execution records | Request/run/attempt identifiers, parent-child operations, lifecycle events, acceptance evidence | Which existing workflow provides the most complete first history? |
| Existing usage/telemetry sources | Provider counters, call identity, timestamps, cumulative-versus-delta semantics | Which source is authoritative for each metric, and where is coverage incomplete? |

The context seed reports a diagnostic comparison of 834,771 versus 578,622 input tokens, or 44.27% more for Torc. Carry it as a **seed-reported historical finding pending source verification**, not a new Tuan result or a verdict on other workloads. Record stale/conflicting documentation in the inventory; do not rewrite the old study during kickoff.

Select the first workflow by evidence quality: stable lineage, provider usage, context references, and explicit completion checks. Prefer a file export or existing event interface over installing instrumentation. If an essential field is absent, document the gap and add the smallest producer-side extension in a separately scoped change.

## Proposed repository and runtime

Use Python with a small installable package, SQLite, and pytest as the provisional baseline. Confirm the supported Python floor against the selected integration and available test host during discovery. Numerical predictors run locally; they do not require an LLM call per event.

Proposed files, created during implementation rather than by this planning task:

```text
README.md                         purpose, quickstart, measurement limits
AGENTS.md                         concise project-specific working rules
pyproject.toml                    package, runtime floor, test configuration
.gitignore / .gitattributes        exclude operational data; normalize text
docs/inventory.md                 verified reuse/extend/build inventory
docs/contracts.md                 events, accounting, forecasts, findings
docs/evaluation.md                cohorts, splits, metrics, decision rules
docs/divergences.md               ideal/practical differences and revisit triggers
src/tuan/
  cli.py                          import, replay, forecast, report entrypoints
  contracts.py                    versioned records and validation
  adapters/                       one verified source adapter initially
  store.py                        event ledger and rebuildable projections
  accounting.py                   normalized metrics and allocation ledger
  checkpoints.py                  bounded evidence snapshots
  forecasting.py                  baseline and challenger interface
  evaluation.py                   outcome joins and chronological scoring
  reporting.py                    JSON and Markdown output
tests/fixtures/                   synthetic or explicitly sanitized evidence
tests/                            contracts, replay, accounting, end-to-end flow
```

These are subsystem boundaries, not a requirement for one abstraction per file. Keep related code together until it earns separation. Observer/advisory modules are added only in their later milestone.

Target the portable core and CLI on macOS `applemac`, Windows `4070pc`/`3060pc`, and Linux `lugos-host`. Use macOS for the first local acceptance run; verify Linux and Windows separately before claiming support there. Share schemas and Python logic; diverge only for virtual-environment activation, paths, file permissions, and locking behavior. Document POSIX and PowerShell quickstart lanes and test paths with spaces and interrupted imports on each supported platform. No service deployment is part of this release.

Before adding CI, verify repository visibility and runner availability. Private-repository jobs use `[self-hosted, lugos-host]`; if no such runner is registered, ask Matt before adding CI and keep local checks available. Do not substitute a hosted runner. Public repositories use hosted runners. Any new hosted Windows/macOS checks must be manual or scheduled at most weekly, never triggered by pushes, pull requests, or tags.

## Evidence, storage, and interface contracts

### Storage choice

| Option | Value | Limitation | Plan |
| --- | --- | --- | --- |
| JSONL files alone | Simple exports and streaming replay | Application must handle indexing, deduplication, and transactional updates | Use for interchange and sanitized fixtures. |
| SQLite event ledger plus derived tables | Local transactions, unique constraints, queryable history, low operating burden | Single-writer design; unsuitable for sharing a live database between hosts | Default for the first release. |
| Shared database/event service | Potential multi-host collection and concurrent access | Adds deployment, credentials, availability, and migration work | Revisit only after measured need. |

The SQLite event ledger is the canonical local record. JSONL is an import/export format, not another independent authority. Derived tables carry transformation versions and can be rebuilt from retained events. Keep databases outside the checkout and live synchronization folders; transfer exports between machines, not a running SQLite database.

Ingest and projection updates must be transactional or recoverable from the committed ledger. Re-importing the same record is idempotent. A duplicate identifier with different content is an explicit conflict, never a silent overwrite. Corrections append a superseding record; original forecasts and observations remain inspectable.

Retain useful, redacted operational events for at least 90 days by default, with longer retention configurable. Set a storage quota and warn before it threatens that window; do not silently discard evidence to satisfy a quota. Retention applies to local exports and backups as well as the database. Reports state which evidence has expired and which replay claims remain possible.

Redact and allowlist fields **before any persistence**, including rejection logs. Default to metadata and access-controlled references; exclude secrets, credentials, hidden reasoning, and unrestricted prompts, responses, or memory contents. Payload capture requires an explicit purpose and opt-in retention class. Restrict local access to the operator account using platform-appropriate permissions. Authorized privacy deletion must purge applicable source records, projections, exports, and managed backups; retain only a non-sensitive deletion receipt. Append-only semantics do not override deletion.

### Minimum records

| Record | Required information |
| --- | --- |
| Event envelope | Event/schema/source versions; stable event ID and source deduplication key; event time and ingestion time; source sequence when available; event type; task/run/attempt/session/agent identities; parent operation; host and process-start identity plus PID; provenance reference; redaction status. |
| Task/run | Request reference and version, task family/repository, acceptance criteria and versions, model/provider/settings, lifecycle state, lineage, context strategy/version. Missing links are explicit. |
| Usage observation | Provider call ID, accounting scope and role, reported metric categories/units, delta or cumulative semantics, measurement provenance/quality, source revision, available cost/billing data. |
| Checkpoint | Immutable ID; available-through ingestion boundary and source positions; task/spec/plan revisions; code commit plus patch identity for uncommitted work; context references; available tests/events/usage; feature snapshot/version. |
| Forecast | Target and units, checkpoint/cutoff, prediction time, estimator/training-data versions, feature snapshot, point/range or unavailable status, interval method, assumptions, inference usage/latency. Revisions link to prior forecasts. |
| Outcome | Completion/failed/abandoned/in-progress status; acceptance evidence and assessed revision; cost-so-far or final cost; quality assessment; outcome-known time; later rework as a linked update. |
| Finding, added later | Claim, affected requirement/location, evidence references, inspected revision, expected consequence, suggested verification, observer version/cost, validation and advisory disposition. |
| Intervention, added later | Finding/policy reference, what advice was shown and when, decision/actor, action, policy revision, result, and rollback reference. |

Use UTC timestamps and retain source clock quality. A PID alone is not an identity. Preserve known parent-child relationships; do not invent them from proximity in time. Use sequence/lineage evidence where clocks disagree. Unknown schema versions and malformed records must fail visibly or enter a sanitized quarantine with reasons, without corrupting accepted history.

Late records may improve later reports but must not enter an earlier checkpoint's evidence set. A source event timestamp earlier than a prediction is insufficient if the record was unavailable when the prediction ran. Retrospective replays must be labeled; never manufacture a prospective forecast by backdating one.

Planned interfaces:

- Adapter: source records → validated normalized events with provenance and import coverage.
- Projector: retained events + transformation version → reproducible analytical views.
- Predictor: checkpoint + explicitly bounded training snapshot → independent forecast or explained unavailable result.
- Reporter: versioned views + report scope → machine-readable JSON and readable Markdown.
- Later observer: checkpoint + bounded evidence → revision-specific findings, with no authority to modify the task.

Compare predictors on the same target, checkpoint, and information cutoff. Save each original output before exposing any peer output. Physical parallel execution is optional; independence and comparable inputs are the requirements.

## Measurement and evaluation rules

### Accounting

- Preserve reported input, output, cache-read/cache-write categories and provider definitions. Cache categories may be subsets of input, so do not add all counters together. Keep estimates and missing fields separate from measured values; unknown is not zero.
- Choose an authoritative call-level source and reconcile parent aggregates against it. Prevent duplicated calls, cumulative-counter snapshots, retries of ingestion, and parent/child summaries from charging the same work twice. Counter resets are explicit discontinuities, not negative consumption.
- Include failed/abandoned attempts, retries, child work, context preparation, retrieval, summarization, maintenance, and observer/reporting overhead. Report execution and observer roles separately, then an explicit whole-workflow total where coverage allows it.
- Keep reusable preparation in an allocation ledger: direct cost plus allocated shares per task, with an identified allocation rule and cohort. Never charge both the full shared preparation and its allocated shares in the same total. Show unallocated overhead.
- Keep tokens, estimated monetary cost, actual billing, subscription allowances, and human effort distinct. Monetary estimates require a dated rate/source; do not infer actual credits or bills from token counts. Do not treat token counts across model/tokenizer families as interchangeable effort units.
- Report wall-clock elapsed time separately from observed active work and user/provider/build waits. Overlapping child durations cannot be summed into wall time. Unobserved waiting or human effort stays unknown.
- Pair usage with completion and quality. In-progress or abandoned cost-so-far is not the final cost of accepted completion; keep these runs in the history and failure reporting without treating them as cheap successes.

### Forecasting

Start with total reported input and output token forecasts as separate targets. Add remaining-usage forecasts at meaningful checkpoints once total-usage accounting is reliable; define remaining usage relative to usage known at that checkpoint and handle late accounting corrections explicitly. Defer elapsed-time and failure-risk models until their labels are dependable.

The baseline is the median and empirical range from compatible earlier tasks. The first challenger is a small regularized regression using a few available features, such as task family, model route, starting context size, and repository characteristics. Keep difficulty as explicit features initially rather than inventing one additive score. Avoid a complex model registry or training service.

With no usable history, emit `insufficient_history` and the evidence needed to improve it. A documented broader-cohort fallback is allowed only when its compatibility and limitations are visible. Synthetic data can prove the model interface works; it cannot prove forecasting quality. A challenger remains unavailable when training or validation support is inadequate.

Save predictions before outcomes, including when outcomes arrive late. Split chronologically and keep an entire task lineage on one side of a split. Cross-repository performance claims need held-out repositories. Freeze training snapshots before evaluation; a historical replay model trained on later tasks cannot count as an honest historical forecast.

Score error, signed bias, underestimation magnitude/rate, range coverage and width, inference overhead, and performance by relevant cohort. Label empirical ranges as uncalibrated until held-out coverage supports a stated level. Track recent error and changes in model, harness, context policy, or task mix before adding automatic drift detectors.

Define promotion criteria before scoring a candidate: required improvement, tolerated underestimation, interval coverage, quality/coverage constraints, and overhead budget. Set data sufficiency from usable independent task lineages, cohort representation, and uncertainty around those decisions, not an arbitrary number of days. If estimates are too uncertain, retain the baseline and keep collecting. Ensembles, online learning, automatic retraining, and automatic promotion wait for evidence that they help.

### Attribution and observer evaluation

Begin with descriptive measurements. Then permit explicitly labeled observational comparisons with declared adjustment/matching criteria and remaining confounders. Controlled studies come later, with a written protocol specifying treatment, baseline, assignment unit, comparable outcomes, model/tools/settings, cache state, context versions, and included overhead.

For an approved randomized trial, assign at a task-lineage level that avoids contamination and record assignment before execution. Keep quality, success, latency, and user effort alongside usage. Preserve unsuccessful and abandoned runs and report their effects on interpretation. Avoid claiming general benefit from a single diagnostic.

`Forecast − actual` is forecast error, not context savings. A descriptive difference between groups is not automatically a causal effect. Report positive, negative, and inconclusive results under the same rules. Do not silently inherit or waive the frozen Torc study's requirements; explain which gates match each new claim.

Evaluate semantic findings by confirmed additional findings, false alarms, missed issues on an adjudicated sample, useful warning time, verification cost, and interruptions. Repeated reads or failures are signals to inspect, not proof of a loop. Evidence and reproduction outweigh majority agreement. If advice fixes a defect, retain its original supporting evidence and action history rather than relabeling it a false alarm because the defect disappeared.

## Implementation milestones

Execute M0 → M1 → M2 as the first release. Later milestones are conditional increments, not prerequisites for making the initial history useful.

### M0 — Verify sources and establish the repo contract

**Outcome:** one selected workflow and an implementable, bounded contract.

Inspect the candidate sources above; write the inventory, contract definitions, evaluation outline, and initial divergence register. Select the first adapter and supported runtime. Add the package/test scaffold, README, concise repo instructions, ignore rules, and line-ending policy. Record the selected platform matrix and permitted CI path.

**Files:** root scaffold, `docs/inventory.md`, `docs/contracts.md`, `docs/evaluation.md`, `docs/divergences.md`, `tests/fixtures/`.

**Acceptance:** obtain a sanitized sample covering request, context, call usage, lineage, and outcome; identify missing fields and ownership without claiming unavailable measurements. Validate the package import and fixture schema locally. A synthetic fixture supports development when real evidence is incomplete, but does not satisfy real-workflow acceptance.

**Dependency/compatibility:** no prerequisite beyond proceeding with implementation. Do not alter producer schemas or original studies merely to make the fixture fit.

### M1 — Build a replayable history with trustworthy accounting

**Outcome:** ingest the selected export into a local ledger and rebuild the same analytical result.

Implement validation, redaction, provenance, idempotent imports, versioned projections, checkpoints, accounting, and outcome linkage. Add explicit coverage and incomplete-record status. Implement the initial retention/deletion path for managed records so the storage contract is real.

**Files:** `contracts.py`, `adapters/`, `store.py`, `accounting.py`, `checkpoints.py`, related contract/replay tests.

**Acceptance:** a focused suite covers duplicate import, conflicting duplicates, cumulative counters, cache subsets, parent/child overlap, failed attempts, missing metrics, late events, clock skew, interrupted ingestion, unknown schemas, and redaction before persistence. Rebuild into an empty database and compare normalized records/totals. Verify authorized deletion cannot be undone by replaying a managed export. Reconcile one actual task's totals to its source, with any gap visible.

**Dependency/compatibility:** depends on M0's adapter/accounting definitions. Keep the source export intact; migrations and rebuilds operate on Tuan-owned storage. Benchmark collection/replay time and storage size rather than assuming overhead is negligible.

### M2 — Deliver forecasts and one understandable report

**Outcome:** complete the first useful release without a service or autonomous intervention.

Add the historical baseline, local regression challenger, immutable forecast records, chronological evaluation, and JSON/Markdown reporting. Expose CLI operations for import, replay, forecast, and report. Run both predictors from one checkpoint; record forecasts independently in shadow mode. Include forecast production and reporting overhead.

**Files:** `forecasting.py`, `evaluation.py`, `reporting.py`, `cli.py`, end-to-end fixture/test, README walkthrough.

**Acceptance:** from a clean environment, follow the README to reconstruct a completed task's evidence, forecasts, outcome, quality, and observation cost. Re-import/replay produces equivalent results. Verify no feature, training record, or outcome after the cutoff leaks into a forecast. Exercise cold start, delayed outcomes, aborted tasks, and a changed context/code revision. Demonstrate both predictor paths using earlier disjoint history, or clearly labeled synthetic data when necessary.

For the real-workflow acceptance, observe one new task through an existing export/checkpoint interface and save forecasts before completion. Where training history is insufficient, preserve an explicit unavailable result; do not backfill a prediction. Run no additional coding workload solely to manufacture results without separate authorization. Report which parts are replay-tested and which have prospective evidence.

**Dependency/compatibility:** depends on M1. Keep output schemas versioned. No champion replacement or claim of savings is needed to pass. The first-release stopping point is a reproducible walkthrough and report, with accounting coverage and forecast readiness made explicit.

### M3 — Accumulate useful operational history

**Outcome:** routine authorized use supplies better evidence with measurable overhead.

Add one lightweight event/export integration only if manual import is insufficient. Prefer existing lifecycle hooks; do not introduce a background service by default. Capture meaningful checkpoints: plan ready, coherent patch, repeated failure, scope change, compaction/handoff, and completion. Add simple repetition/failure signals and continue baseline/challenger scoring.

**Files:** the selected adapter, checkpoint trigger logic, a small rules module, evaluation/report extensions.

**Acceptance:** source-to-ledger reconciliation survives restart and reconnect without duplicate charges; collection failure is visible and does not take over execution. Measure missed events, ingest latency, storage growth, and overhead against a declared operating budget. Verify configured retention with controlled timestamps. Establish prospective evaluation cohorts before judging predictors.

**Dependency/rollout:** depends on the M2 walkthrough. Start with one opted-in workflow; broaden adapters only after each passes the common contract. This milestone does not require waiting 90 days to release.

### M4 — Add one bounded semantic observer

**Outcome:** test whether a reviewer contributes useful evidence beyond numerical rules.

Start with unsupported completion claims: compare acceptance criteria and available checks against the inspected revision. Dispatch at completion-related checkpoints under an explicit invocation/cost budget. Persist findings separately from forecasts. A single advisory stage deduplicates findings, checks currency, records validation, and chooses what to surface. Default to shadow reporting before allowing advice to influence work.

**Files:** a narrowly scoped observer and advisory module, finding contracts, revision/cutoff fixtures, observer scorecard.

**Acceptance:** detect seeded unsupported claims, avoid a supported claim, preserve a stale result against its original revision, and require revalidation when relevant code changes. Measure confirmed contribution, false alarms, warning time, overhead, and interruptions. If execution is needed for verification, use an isolated checkout rather than sharing the active writer's workspace.

**Dependency/rollout:** requires reliable M2 records and an explicit model/cost choice before paid calls. Add other specialists only when they address a measured gap. The execution system remains responsible for any response to advice.

### M5 — Evaluate context interventions and recommendations

**Outcome:** answer a specific context-impact question at a stated evidence level.

Choose one intervention and comparison, such as an existing context selection policy versus its agreed baseline. Write the attribution protocol, cost-allocation rule, quality criteria, and stopping/reporting rules before examining the comparison. Start with descriptive or observational results; launch matched or randomized trials only when separately authorized and feasible. Keep observer advice fixed or explicitly treat it as another intervention.

**Files:** one study protocol under `docs/studies/`, reusable analysis in `evaluation.py`, report extensions, immutable experiment metadata.

**Acceptance:** reproduce reported group totals, effect estimates, uncertainty, exclusions/missingness, and quality outcomes from retained evidence. Include preparation, retries, observer, and maintenance costs. Report a measured disadvantage as readily as a benefit; record the smallest next measurement that would reduce uncertainty.

**Dependency/rollout:** depends on a usable operational cohort and verified treatment metadata. No automatic context or memory changes follow from a favorable report.

## Report contract and example

Every report names its period, task/cohort, baseline, accounting scope, evidence coverage, outcome quality, and source/transform versions. Lead with the strongest supported observation. Distinguish **observed**, **estimated**, **experimental**, and **unknown** values; state each material limitation once and name a useful next measurement.

The following is entirely illustrative, not a result from this repository:

| Question | Illustrative answer |
| --- | --- |
| What happened? | One accepted task consumed 60,000 reported input tokens and 8,000 output tokens across all included roles. Acceptance checks passed for the recorded revision. |
| What did we expect? | Baseline input forecast: 70,000, empirical range 50,000–100,000. Challenger: 66,000, range 48,000–90,000. Ranges are not yet calibrated. |
| What was the forecast error? | Baseline overpredicted input by 10,000; challenger by 6,000. This is one task, not evidence of a model winner. |
| What overhead was included? | Of the 60,000 input tokens, 5,000 were context preparation and 2,000 observation/reporting. These are subsets of the total, not extra charges. |
| What about cache and time? | Source reports 20,000 cached input tokens within the input total; elapsed time is 25 minutes, with 5 minutes of recorded waiting. Human effort is unknown. |
| What context-related difference was established? | No comparable treatment baseline is available for this task; the context effect is unknown. The forecast error does not estimate it. |
| What should happen next? | Keep this task in forecast scoring; collect a comparable task under the specified alternative context policy before estimating an observed group difference. |

Once comparison evidence exists, add the observed difference and, separately, any supported context-effect estimate with its assumptions and uncertainty. Actual billing remains unknown unless an authoritative billing source supplies it.

## Deferred choices, risks, and divergence register

| Ideal capability or risk | Kickoff choice | Revisit trigger |
| --- | --- | --- |
| Broad integration coverage | One adapter with explicit missing fields | First workflow is reproducible and another source adds useful coverage. |
| Shared multi-host collection | Local single-writer ledger and portable exports | Measured concurrency/collection needs exceed this design. |
| Accurate calibrated predictions | Baseline, one challenger, explicit cold-start status | Independent held-out data supports the declared accuracy/coverage decision. |
| Multiple specialized observers | Rules first, then one completion-evidence reviewer | Additional confirmed value justifies more cost and interruptions. |
| Causal context-effect estimates | Descriptive reports first | A specified comparison, outcome criteria, and adequate design are available. |
| Complete operational evidence | Metadata and sanitized references | A concrete analysis needs more content and its access/retention is approved. |
| Replay after schema changes | Versioned transformations and retained events | A schema migration must pass replay parity before adoption. |
| Replay after deletion/expiry | Honest partial coverage | Never reconstruct erased content; revise dependent claims instead. |
| Automatic improvement | Recommendations only | A separately approved bounded policy, independent evaluation, and rollback are ready. |

Future “self-healing” must specify the detector, the exact allowed action, quality checks, independent success measure, audit record, rollback, and stop conditions. Optimizing usage alone or allowing the evaluator to rewrite its own evidence would not satisfy that contract.

Decisions to resolve at their point of need:

- **During M0:** exact first workflow/export, source owners, runtime floor, operator data location, and how accepted completion is established. Resolve from evidence where possible; ask Matt only when source access or intended workflow remains ambiguous.
- **Before operational collection:** choose storage quota, retention above the 90-day default if needed, and any payload opt-in. Metadata-only is the default.
- **Before model promotion:** agree acceptable underestimation, empirical coverage, and overhead bounds. Until then, display the baseline and keep challengers in shadow mode.
- **Before semantic calls or intervention trials:** choose the narrow scope, spending limit, and experiment conditions. No such execution is included in this plan-only task.

## Execution and final verification

Implement the first release directly in coherent batches because storage, accounting, and forecasts share contracts. Independent adapter work becomes useful only after those contracts stabilize; give each adapter a disjoint write area and a shared conformance suite. Do not create one agent or reviewer per module.

At the end of M2, perform one consolidated self-review and run fresh contract, accounting, replay, and end-to-end checks. Inspect the final diff for unrelated edits, sensitive fixtures, accidental services, and unsupported capability claims. Document platform checks that passed and those still pending. Broaden review only for a material unresolved integration or data-integrity risk.

Coverage of the seeds is explicit: inventory and ownership → M0; replay/retention/accounting → M1; immutable forecasts, competition, evaluation, and reports → M2; proactive history and rules → M3; semantic observers and advisory handling → M4; context attribution and controlled experiments → M5. The ideal concept and divergence register remain alongside the practical milestones.

**Recommended next action:** when implementation is requested, start M0 and carry through M2 to produce the first replayable real-workflow report. Do not begin a collector deployment, paid review, experiment campaign, or autonomous adjustment as part of repository kickoff.

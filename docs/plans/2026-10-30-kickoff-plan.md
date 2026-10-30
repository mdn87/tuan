# Tuan kickoff plan

Date: 2026-10-30. Status: proposed. Owner: Matt.

## What Tuan is

Tuan is a standalone Lugos project for agent-work measurement, context-impact
estimation, and usage forecasting. It keeps a trustworthy history of what we
expected, what actually happened, and what context was supplied, then uses
that history to improve estimates and to evaluate context strategies.

It runs alongside Sulis (the durable memory layer) and consumes Torc's
continuity and context evidence. Torc is an integration and the source of the
prior research, not the owner. Tuan is not another memory store and does not
orchestrate execution; that stays with Sulis and Autowork.

The name is Tuán mac Cairell, who preserves and recounts history across
successive transformations. The package and submodule are both `tuan`.

## Inputs

- `context-impact-mamagement.md`: the shaping conversation and the revised
  kickoff prompt (sections 1 to 7 plus deliverables). Phase 0 renames it.
- `predictive-analytics.md`: predictor and observer architecture, checkpoints,
  evaluation design, and the build-first list.
- Torc research on `origin/master` at 84c0ec8 (2026-10-22). The local torc
  submodule checkout is 16 commits behind at 26950f6 and does not yet have
  these files, so read them from origin or fast-forward with `git_sync.py`:
  - `docs/plans/2026-10-21-measured-efficiency-plan.md`
  - `docs/plans/2026-10-21-efficiency-execution.md`
  - `experiments/session-continuation-001/diagnostic-results.md`
- Frozen finding to preserve unchanged: in the one diagnostic workload where
  both portable approaches passed, Torc used 834,771 input tokens against
  578,622 for the plain handoff, 44.27% more. It is a result for that
  comparison, not a verdict on every context strategy or workload.

## Responsibilities

| Capability | Question it answers |
|---|---|
| Observation | What context, usage, work, and outcomes occurred? |
| Forecasting | What usage and effort should we expect before and during a request? |
| Attribution | How much difference can reasonably be attributed to context selection, compression, retrieval, continuity, or memory use? |

These stay distinct. Accurate prediction is not proof of causation. A smaller
starting context is not necessarily lower total effort; more initial context
may prevent repeated investigation, mistakes, or lost requirements, or may
simply add overhead. Both get measured.

Automatic memory changes, context tuning, and agent routing are disabled in
the initial design and stay disabled until Phase 5 defines their safeguards.

## Ground rules

Reporting

- Lead with the strongest supported measurement or estimate. Label every
  quantity observed, estimated, experimental, or unknown. Name the baseline
  and the reporting period.
- State each material limitation once, then name the smallest next
  measurement that would reduce it. Never substitute "cannot prove savings"
  for the evidence already in hand.

Evidence

- Append-only, schema-versioned, replayable records. Append-only prevents
  silent rewriting; it does not prevent authorized privacy deletion.
- Every event carries event time and ingestion time, request/task/run/attempt
  identifiers, parent links, provider/model/settings, context-policy version,
  and process identity including host and process start time (a PID alone is
  reused). Handle clock skew, duplicates, late arrivals, retries, restarts,
  and incomplete records explicitly.
- No secrets, credentials, hidden reasoning, or unrestricted prompt, response,
  or memory contents are persisted. Redaction happens before persistence.
  Payload capture is opt-in per class. Collection overhead is measured, not
  assumed free.

Accounting

- Input, output, and cache categories are stored separately with
  provider-specific definitions. Cached counts can be subsets of input totals;
  never sum reported numbers blindly.
- A missing measurement is unknown, not zero.
- Observer usage is separate from coding usage. Elapsed time is separate from
  active work (waiting on a user, a provider queue, or a build is
  identifiable).
- Failed and abandoned work, retries, child calls, context preparation,
  retrieval, summarization, and maintenance all count. Reusable preparation
  cost is allocated once, never charged twice.
- Success and quality are tracked alongside usage. A cheap failure is not a
  savings success.

Forecasts and evaluation

- A forecast is stored before execution with its timestamp, as-of data cutoff,
  estimator version, feature snapshot, predicted range, assumptions, and the
  cost of producing it. Revisions sit beside the original; nothing is replaced
  with hindsight.
- Initial total-work estimates and in-flight remaining-work estimates are
  different targets with different evidence.
- Evaluation is chronological. Task lineages stay together; later tasks and
  held-out periods are the test set; cross-repository claims need a separate
  repository holdout. Intervals are scored on coverage and width. Failed runs
  stay in the dataset.

## Phases

### Phase 0: repo seed

Already done: repository created, the two input docs pushed, `tuan`
registered as a Lugos submodule (mdn87/lugos#682).

1. Lay out the repo the way Torc and Cormac are laid out: `README.md`
   (concept and boundaries), `AGENTS.md` with a `CLAUDE.md` pointer, `LICENSE`
   matching the sibling projects, `pyproject.toml` (Python, like Torc and
   Autowork), `docs/{research,plans,decisions}/`, `schemas/`, `src/tuan/`,
   `tests/`.
2. Add `.gitignore` with `.lugos-audit/`, as the lugos root, torc, and cormac
   do.
3. Move the two inputs to `docs/research/` and rename
   `context-impact-mamagement.md` to `context-impact-management.md`.
4. Decide repository visibility (D1). The repo is currently public.

Exit: the repo is navigable from its README, `lugos-audit quick` is valid, and
the lugos submodule pointer is bumped with `scripts/git_sync.py bump`.

### Phase 1: discovery and inventory

Planning only. Produce `docs/inventory.md`: a reuse / extend / build table
with source paths, current state, and owner for each item, verified from
source and deployment evidence rather than plans.

Sources to inventory:

- Torc: continuation accounting, context projection, the diagnostic and
  report scripts under `experiments/session-continuation-001/`,
  `docs/attempt-accounting.md`, `docs/evaluation-plan.md`.
- Sulis: what versioned metadata and identifiers memory capture and recall
  already expose.
- Autowork: campaign request envelopes, run identities, `logs/`.
- lugos-audit receipts and the `agent-journal` and `verification-evidence`
  toolset skills as outcome evidence.
- Claude Code and Codex session transcripts and usage on each inventoried
  host.
- lugos-link inventory for stable host identity.

Also in this phase:

- Flag stale or conflicting documentation that leads agents to repeat
  inaccurate limitations (the Torc README offline summary versus the 10-21
  measurement plan is the known example).
- Classify each gate from the frozen Torc study as study-specific or adopted
  here, with the reason. Do not silently waive old gates or impose all of
  them.

Exit: every "reuse" row has a verified path and state; stale docs are listed
with the correction each needs.

### Phase 2: design package

- `docs/concept.md`: plain-English concept and ownership boundaries. An ideal
  design beside the practical one, with `docs/divergence-register.md` listing
  each compromise and when to revisit it.
- `schemas/`: event record; checkpoint snapshot (task definition, code
  revision, recent events, test results, usage so far); forecast record
  (target, prediction time and checkpoint, model and feature versions,
  estimate and range, cost to produce); finding record (claim, requirement or
  code location, evidence, expected consequence, suggested verification,
  revision inspected).
- Identity: stable task/run/attempt ids with parent links; host plus process
  start time. Evaluate OpenTelemetry trace and span conventions for
  propagation across processes and machines (tracing context is a separate
  concept from the prompt context Torc manages).
- Storage: compare an append-only JSONL log with SQLite or DuckDB analytical
  views before proposing any service. Default 90-day operational retention,
  configurable longer, rebuildable views, an authorized deletion path.
- Integration contracts: Sulis (read versioned metadata, never write
  memories, never duplicate contents), Torc (continuity and context evidence
  source), Autowork (checkpoint emission now; consumer of findings later).
- Redaction, access, payload opt-in, and retention policy.
- Measurement definitions, the attribution plan (named treatments, fair
  baselines, a progression from descriptive measurements to labeled
  observational estimates to matched or randomized comparisons, confounders
  listed), and the forecasting plan (baseline, one challenger, shadow mode,
  chronological evaluation, drift detection, evidence-based data-sufficiency
  criteria rather than a waiting period).
- One example report using clearly labeled illustrative numbers.

Exit: Matt has reviewed the package and decisions D1 to D6 are recorded in
`docs/decisions/`.

### Phase 3: first milestone

First code. Needs approval of the Phase 2 package. The smallest useful loop:

1. One instrumented workflow (D2) emitting checkpoint events: plan ready,
   coherent patch produced, repeated test failure, scope change, handoff or
   compaction, completion claimed.
2. Append-only event store with replay into versioned analytical views.
3. Two forecasts of total and remaining usage recorded before the outcome, in
   shadow mode: a historical baseline (median and range from similar prior
   tasks) and one challenger (a small regression over task features).
4. One understandable report per task and a period rollup, following the
   reporting rules above.

Acceptance test: take one completed coding task and reconstruct what each
predictor knew at each checkpoint, what it predicted, what actually happened,
and what the observation itself cost. The record must be dependable across
the number of tasks set in D6 before any champion/challenger comparison
counts.

Constraints: no collectors installed as services, no paid or model
experiments, no change to Sulis, no autonomous adjustment.

### Phase 4: observers

After Phase 3 evidence exists, add observers one at a time and measure the
added value of each before adding the next.

- Loop detector as rules first: repeated reads of unchanged files after a
  context reset, a test failure repeating without a changed assumption,
  reverting and recreating similar patches, diff growth while acceptance
  criteria stay unresolved, growing effort spent recovering earlier
  information, completion claimed while an explicit requirement has no
  verification. Each is a reason to inspect, never an automatic failure.
- One LLM semantic reviewer with one narrow job (requirement drift or
  unsupported completion claims) at selected checkpoints. Isolated checkout if
  it runs tests. Its original result is recorded before it sees any other
  observer's output.
- A single advisory stage deduplicates findings, revalidates them against the
  current revision, and decides what to surface.
- Scorecard: confirmed findings, missed problems, false alarms, warning lead
  time, and cost. Agreement rate is not a metric.
- Ensembles only after individual predictors' error overlap has been measured.

### Phase 5: attribution study and improvement loop

- Define the exact context treatments compared, consistent completion and
  quality criteria, how difficulty, model route, and repository are handled,
  which preparation, review, retry, and recovery costs count, and whether
  observer advice may influence the run. Randomize assignment where practical.
- Keep three questions separable: does improved context help, do observer
  interventions help, does the combination justify its total cost.
- Progression: reports, then recommendations, then human-approved
  context-policy experiments. Any later automatic adjustment needs bounded
  authority, versioned policies, quality safeguards, an audit trail,
  rollback, and evaluation data it cannot influence. "Self-healing" is
  defined here as: what detects a problem, what is allowed to change, how
  improvement is verified, and when the system must stop or ask.

## Decisions needed

| Id | Decision | Default if none |
|---|---|---|
| D1 | Repository visibility (currently public) | Public for code; operational data never enters the repo |
| D2 | First instrumented workflow: Claude Code sessions via hooks on applemac, Autowork campaigns on 4070pc, or Codex | Claude Code on applemac, where the hooks already run |
| D3 | Where the event store lives: per-host files collected to lugos-host, or lugos-host only | Per-host JSONL, collected to lugos-host |
| D4 | Retention beyond 90 days and the payload opt-in default | 180 days, payloads off |
| D5 | Import the Torc trial data as the first historical dataset | Yes, labeled as the frozen study |
| D6 | Data-sufficiency threshold for the first champion/challenger comparison | Set from the inventory, not a fixed count |

## Risks

- Double counting cumulative counters, cache subsets, or parent and child
  calls. Mitigation: per-provider accounting definitions in `schemas/` before
  any ingestion.
- Future information leaking into features or evaluation. Mitigation: as-of
  cutoff on every forecast record; grouped chronological splits.
- Stale documentation causing agents to repeat "no evidence". Mitigation:
  Phase 1 stale-doc list; the reporting rules in this plan.
- Observers interfering with the working directory they measure. Mitigation:
  isolated checkouts; read-only observers by default.
- Observer cost exceeding its benefit. Mitigation: scorecard with cost before
  any second observer is added.
- Process identity reused across machines. Mitigation: host plus process
  start time is mandatory in every event.
- Public repository plus operational data. Mitigation: D1; data paths
  ignored; redaction before persistence.

## Out of scope for now

Ensembles, automatic adjustment, memory edits, agent routing, dashboards, and
any long-running service.

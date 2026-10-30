# Architectural-Drift Transcript: Integration Proposal

**Status**: cross-reviewed and integrated. Adversarial `codex-astra` cross-review run `20261030-060647-2-cross-review` returned 11 findings (F1–F11); all were applied here (Section 7) and the refined proposals P1–P8 were integrated into `docs/plans/2026-10-30-combined-kickoff-plan.md` (the kickoff plan) on 2026-10-30.
**Date**: 2026-10-30
**Source under evaluation**: `docs/architectural-drift.md`, an untracked pasted chat transcript (175 lines, no headings). Part A (L1–65) is an assistant answer on tying architecture decision records (ADRs) to the tracking system. Part B (L79–89) is Matt's note on deferring archive work, keeping logs about three years, a weekly rescan, and weighting recent evidence 60–70%. Part C (L109–176) is the assistant's refinement and correction of Part B.
Line references to the kickoff plan below are to commit `82a8d81`, the revision reviewed.

---

## 1. Recommendation

**Tie the transcript into the kickoff plan. Do not build it as a parallel project.**

1. Most of the transcript's substance is already in the plan (Section 3). What remains is eight bounded amendments to existing sections and milestones (Section 4): no new milestone, no new storage system, no service, and no enforcement authority.
2. A parallel "drift" project would need its own record contracts for the same runs. The plan's single per-host ledger, provenance rules, and append-only semantics argue for common contracts now; they do not require implementing every consumer of those contracts in the first release (F11).
3. The transcript conflates two problems with different owners. **Decision drift** (the build departing from accepted decisions or the ideal concept) is a decision-record and observer concern. **Score staleness** (estimates and effectiveness scores going stale as conditions change) is a forecasting and evaluation concern. One parallel project would have no coherent boundary; the plan already has a home for each.
4. The transcript's own strongest advice, "finish the first useful feature, preserve the evidence now, defer historical rebuilding", is the plan's existing M0 → M2 shape.

**Milestone disposition after review (F11)**: P1 and P2 provenance contracts land in M0–M2 with focused M2 validation; P3 is documentation now; P4's record format is M0; P7's retention-class decision is taken before the first M1 import with its fixture in M1; **P5 (recency weighting) and P6 (`tuan refresh`) move to M3**, where their cost is measured in the milestone that introduces them; P8 is M4 and later. The M0–M2 first release does not gain a CLI verb or a scoring policy.

What this proposal does **not** do: it does not draft the D1–D6 decision texts (parked by Matt), does not add a scheduler or background service, does not add an enforcement gate on patches, and does not reopen D4.

---

## 2. Source Assessment

- **Form**: a raw transcript with interface residue ("Worked for 2m 21s"), KaTeX markup, and no structure. It is source material, not a design note. Its title is misleading: nothing in it defines or measures architectural drift as such.
- **Reliable content**: Part A's four observer responsibilities and the "link each run to the decision revision and the implementation actually used" rule; Part C's four-way distinction between original estimate, recorded actual, recalculated score, and revised estimate; Part C's correction that a revised estimate cannot replace a missing actual and that rescoring does not eliminate drift; the "normalize groups before weighting" catch; the "check what changed" maintenance design.
- **Content to treat as hypotheses, not requirements**: the 65/35 split, the weekly cadence, and the three-year retention figure. The transcript itself labels them starting points or product decisions.
- **Disposition of the file**: keep it for traceability like the other two seed drafts. When M0 task 1 moves the seed docs into `docs/research/`, move this file there as `docs/research/2026-10-30-architectural-drift-transcript.md` with a three-line header (provenance, that it is a transcript, and which proposals below absorbed it). Do not cite it as a design authority.

---

## 3. Already Covered by the Plan (no change proposed)

| Transcript point | Where the plan already has it |
| --- | --- |
| Divergence register: reason for each compromise, revisit condition, concept kept alongside the build | Divergence Register (L248–260); "The ideal concept and divergence register remain alongside the practical milestones" (L302) |
| Forecast before outcome; information cutoff; bounded training snapshot; delayed progressive validation; leakage warning | Forecast record (L82); Predictor interface (L91); comparison rule "same target, checkpoint, and information cutoff… save each original output before exposing any peer output" (L95); Strict temporal cutoffs (L61); M2 adversarial cutoff tests (L182) and prospective verification (L183); Key Risks (L288) |
| Retrospective replays explicitly labeled; never backdate a prospective forecast | Strict temporal cutoffs (L61); M2 retrospective reconstruction (L182) |
| Preserve original predictions; revisions never overwrite | Forecast record "Revisions link to prior forecasts without overwriting" (L82); Append-only and idempotent (L58) |
| Actuals and late corrections kept apart, every label revision preserved | Outcome record (L83); M2 "each score recording the label version it used" (L178) |
| Capture task/run/process identities, model and configuration versions, references to inputs, code, and decisions now | Event envelope, Task/run, Checkpoint records (L78–81); M0 exit criteria sample fixture (L159) |
| Observer has no authority to change the task; findings bound to the inspected revision | Observer interface (L93); Finding record (L84); M4 items 2–4 (L208–210); "the execution system remains responsible for any response to advice" (L212) |
| Separate evaluations for non-comparable task types | Cohorts throughout: task family in Task/run (L79), "performance by cohort" (L178), D6 cohort representation (L273) |
| Retraining and ensembles only after evaluation works | M5 item 3 (L223); promotion criteria defined before scoring (L178) |
| No automatic recalibration or self-adjustment without guardrails, holdout, rollback | L21; M5 governed improvement contract (L224); point-of-need "Before model promotion" (L280) |
| Decisions recorded as ADRs | M0 task 4 "record decisions D1–D6 in `docs/decisions/`" (L157) and M0 exit criteria (L159) |
| "Finish the first feature, preserve evidence, defer historical rebuilding" | First release = M0 → M1 → M2 (L141); M2 outcome "smallest useful end-to-end loop without background services" (L173) |

---

## 4. Proposed Plan Changes (P1–P8), as refined by the cross-review

Each proposal names the transcript source, the gap in the plan at `82a8d81`, the change as integrated, and the milestone that carries it.

### P1. Decision references and observed-in-effect configuration on the run record

- **Source**: Part A L59–65 ("link each relevant task run to the applicable ADR ID and revision… the implementation or configuration actually used… accepting an ADR does not mean every subsequent task immediately uses that design").
- **Gap**: The Task/run record carried `context policy/strategy version` but no decision reference, and nothing said the policy version must be the one *observed in effect* rather than inferred from a decision's acceptance date. Without this, M5 attribution can credit a decision that had not yet taken effect for a run, and the link cannot be backfilled honestly later.
- **Change (Minimum Record Contracts, Task/run row)**: configuration identifiers are recorded as *observed in effect* for the run, with declared-versus-observed provenance flagged and mid-run changes ordered by event sequence, never inferred from a decision's acceptance date; applicable decision references carry a namespaced decision ID, an immutable revision, the lifecycle status at run start, and the applicability source, when a producer or operator supplies them; `unknown` is a valid value.
- **Change (M0 task 2 inventory, per F8)**: a "Decision and configuration identity" inventory item maps each field per host platform as native, operator-supplied, derived, or unavailable, with a representative fixture per class, and M0 fixes the decision-ID namespace, revision representation, applicability sources, historical status evidence, configuration identity scope, provenance flag, and mid-run ordering in `docs/contracts.md`. Population feasibility is **unverified** until that mapping exists; `unknown` remains a valid M1 outcome; no producer schema change is authorized (the M0 selection rule governs that).
- **Milestone**: contracts in M0; populated from M1 adapters where available.

### P2. Four record kinds, a derived Score contract, and defined leakage status

- **Source**: Part C L153–170 (table of four record meanings; "a revised estimate cannot replace a missing actual outcome"; "comparing one estimate with another measures their agreement"; leakage when a predictor is replayed on tasks it learned from).
- **Gap**: The plan had original forecasts, forecast revisions, outcomes with label versions, and labeled retrospective replays, but never named a *recalculated score* or a *revised estimate*, recorded no scoring-rule version, and had no identity or selection rules for superseding scores (F2) or defined leakage statuses (F3). Part B's "you can't get the actuals anymore but you can reassess your estimates to see the difference" is exactly the confusion this prevents.
- **Change (Accounting Rules, new bullet "Estimate and score lineage")**: four record kinds, never substituted for one another: **original forecast**, **recorded actual**, **recalculated score** (a scoring-rule version applied to preserved forecasts and actuals under the Score contract; supersedes without deleting), and **revised estimate** (a newer predictor, training snapshot, or predictor-input transformation replayed against a historical checkpoint; always `retrospective_replay` with a leakage status). Evaluation-only changes rescore unchanged predictions; a change that alters predictor inputs requires new predictions, which are revised estimates (F1). A revised estimate is never scored as prospective accuracy whatever its leakage status and never stands in for a missing actual; comparing two estimates measures agreement only; a missing actual stays `unknown`.
- **Change (new Score record, F2)**: score ID derived deterministically from its binding so repeated evaluation is idempotent; bound forecast ID (or immutable aggregate membership), outcome and label revision, target and metric, scoring-rule implementation and configuration version, weighting-policy ID and version when applicable, evaluation-as-of time, and the superseded score ID. A current view selects the latest score per binding within one scoring-rule and policy series; incompatible series are never merged; the unweighted series is always retained.
- **Change (Forecast record, F3)**: forecast kind (`prospective` or `retrospective_replay`) and, for replays, a leakage status with defined values and evidence: `cutoff_verified` (training-snapshot and feature-snapshot bounds demonstrably at or before the checkpoint's information cutoff), `contaminated` (training or features known to include the task's outcome or later records), `unknown` (provenance unavailable; never upgraded without evidence).
- **Change (M2 item 2 and exit criteria)**: scores recorded under the Score contract with label and scoring-rule versions; fixtures prove that a label correction, a scoring-rule change, and a repeated evaluation run neither duplicate contributions nor erase earlier scores or the unweighted reference; a predictor-input transformation fitted on later outcomes never silently regenerates predictions; replays are exercised with a clean snapshot, outcome-contaminated training, later-record features, and unavailable provenance, and every revised estimate stays excluded from prospective accuracy.
- **Milestone**: M1 contracts, M2 evaluation and exit criteria.

### P3. Divergence resolution rule and trigger lifecycle

- **Source**: Part A L55 ("an agent should not resolve a divergence simply by editing the ideal concept to match what was built"); Part A L43–53 (reconsideration triggers become review proposals).
- **Gap**: The Divergence Register recorded compromises and revisit triggers but did not say what counts as resolving one or how triggers are evaluated.
- **Change (Divergence Register, paragraph under the table; merged with P8's trigger lifecycle per F11)**: a divergence is resolved only by (a) changing the implementation to match the concept, (b) a recorded, deliberate concept change with its evidence and a superseding decision record, or (c) recording it as accepted with its revisit trigger. Editing the ideal concept to match what was built, without that record, is not a resolution. Revisit triggers are evaluated against retained evidence, manually at milestone boundaries until `revisit_proposal` findings exist (M4); a satisfied trigger produces a review proposal, never an automatic change.
- **Milestone**: now (documentation only).

### P4. Decision record format with evaluable consequences and an immutable lifecycle

- **Source**: Part A L25–42 (decision as measurable hypothesis), L24 (proposed vs accepted), L57 (supersede, never edit).
- **Gap**: M0 task 4 named no format. Lugos has two ADR styles: numbered `NNNN-slug.md` with Status/Date/Context/Decision/Consequences (`torc/docs/decisions/`, `codelink/docs/decisions/`) and an `ADR-TEMPLATE.md` adding Milestone and Alternatives Considered (`sulis`, `aeta`). Neither carries evaluable consequences, and a mutable Status line conflicts with "never edit an accepted record" (F9).
- **Change (M0 task 4)**: numbered `NNNN-slug.md` records, IDs namespaced `tuan:NNNN`, with Date, Context, Decision, Consequences, Alternatives Considered, and four evaluable fields: **Expected benefit** (observable; names the Tuan record or view that would show it and the earliest point it could be observed), **Accepted downside**, **Measurement** (what is compared against which baseline, noting that task mix, model, and caching changes can explain a difference), and **Reconsideration trigger** (a condition on retained evidence). Decision content is immutable once accepted and its revision identity is the accepting commit; lifecycle (`proposed`, `accepted`, `superseded by NNNN`) is recorded as dated entries in `docs/decisions/lifecycle.md` and successor links, never by editing an accepted record. A decision applies to a run or inspected revision when it was accepted and not yet superseded at that time; a `proposed` record never binds a run.
- **Milestone**: M0 (format only). **Out of scope**: the D1–D6 texts themselves.

### P5. Recency weighting as an opt-in, versioned scoring policy (M3)

- **Source**: Part B L84–85 (weight recent 60–70%); Part C L122–137 (define "recent", what is weighted, comparability; normalize groups; keep configurable; small recent groups carry visible uncertainty).
- **Gap**: Scorecards had no stated recency policy; the first draft of this proposal made weighted reporting an M2 requirement without a selected policy, a minimum-support rule, or the temporal and membership inputs needed to reproduce a weighted score (F6, F7, F11).
- **Change (M3 item 4)**: an opt-in, versioned **recency-weighting policy** for scorecards, off until a recorded decision selects it; the unweighted score stays the reference and promotion criteria stay unweighted. The policy and each weighted score's evaluation identity fix the window size and basis (outcome-known time, or completed-lineage count with deterministic ordering), the evaluation-as-of time, the eligible cohort and lineage population, the contribution unit, and the label-selection rule; late outcomes and corrected completion times create a new as-of evaluation rather than rewriting an earlier one, so a current reassessment stays distinguishable from a historical-as-of result. Weights are group-normalized so a stated split (such as 65/35) holds regardless of group sizes, which is disclosed as intended; below a declared minimum recent support the weighted score is suppressed as `insufficient_recent_support`; an empty group falls back to the unweighted score with an explicit note.
- **Change (M3 exit criteria)**: tests cover a window-boundary crossing, a late correction, multiple forecasts in one lineage, the minimum-support and empty-group boundaries, and rebuilding the same as-of evaluation after later events arrive.
- **Change (Divergence Register, new row)**: ideal "adaptive recency-weighted estimators" → kickoff "unweighted baseline; opt-in weighted scorecards from M3" → revisit when the weighted/unweighted gap is material across cohorts over the D6 history.
- **Change (point of need, before M3)**: whether to enable the policy is decided with the other pre-collection settings.
- **Milestone**: M3. A recency-weighted baseline variant may enter only as an additional shadow challenger afterwards.

### P6. Change-driven maintenance verb `tuan refresh` (M3)

- **Source**: Part B L83–87 (weekly rescan, short-circuit when nothing changed, automate later); Part C L138–152 (make the job check what changed).
- **Gap**: The plan had `tuan replay` and versioned projections but no idempotent maintenance entry point. The first draft derived pending work from a projection position (which cannot detect scoring missed after a projection advance or an interruption, F4), wrote receipts into a ledger it also required to stay byte-identical (F5), and conflated predictor-input transformation changes with scoring-rule changes (F1).
- **Change (M3 item 3)**: `tuan refresh` derives pending work from durable **evaluation-coverage records** (which forecasts and lineages have been scored under which scoring-rule and policy versions), kept independently of projection progress; new events or newly known outcomes → project and score affected lineages; a changed evaluation-only scoring rule → superseding recalculated scores; a changed predictor-input transformation or predictor version → retrospective revised estimates only when explicitly requested; a crossed recency-window boundary → refreshed weighted scores; nothing relevant → a recorded no-op. Score identities are deterministic so repeats are retry-safe, and coverage progress commits only after a unit's scores are durably written. Receipts (what changed, what was recomputed, versions, duration) are operational records written outside the evidence ledger, and receipt-only changes never count as pending work. It never retrains or promotes a predictor and never modifies ledger originals.
- **Change (M3 exit criteria)**: refresh runtime, storage growth, and no-op cost measured against the declared budget; tests for already-projected but unscored events, interruption between projection and scoring, late outcome corrections, repeated execution, and a no-op that leaves the evidence ledger and projections byte-identical while writing only its operational receipt.
- **Scheduling**: running it from cron on `lugos-host` is an operations decision taken after M3 measures its cost; it is a periodic batch over Tuan-owned projections, not a service, and stays inside the L21 boundary; automatic retraining or promotion remains governed by the M5 contract and the "Before model promotion" decision.
- **Milestone**: M3 (moved from M2 per F11).

### P7. Long-lived compact evidence class alongside D4

- **Source**: Part B L82 (keep logs about three years); Part C L119 ("Three years is a reasonable proposed retention policy for those records. It is a product decision…").
- **Gap**: D4 sets 180 days operational retention with payloads off. The three years applies to compact evidence, not to operational records or payloads. The plan scheduled the retention-class decision before the first M1 import but named no long-lived compact class and had no executable check that mixed retention keeps replay claims honest or that deletion reaches derived records (F10).
- **Change (point of need, before the first M1 import)**: decide whether to define a **long-lived compact evidence class** (original forecasts, recorded actuals and label revisions, recalculated scores, decision and configuration references, identities and versions; no payloads), proposed at three years, distinct from the 180-day operational class (D4) and any payload class, with its class assignment, expiry clock, and dependency behavior defined in the same decision.
- **Change (Retention and deletion rule)**: compact evidence records may belong to a longer retention class than operational records; authorized privacy deletion applies across every class, including superseding scores, projections, and other derived records, and a rebuild or later maintenance run after deletion resurrects nothing.
- **Change (M1 exit criteria)**: the storage benchmark includes the compact class if adopted; a retention fixture with controlled timestamps expires operational records while retaining compact records, then verifies which summaries remain valid, which replay claims become unavailable, and that reports say so; deletion is exercised across originals, revisions, superseding scores, projections, and other derived records.
- **D4**: unchanged.
- **Milestone**: decision before the first M1 import; fixture and benchmark in M1.

### P8. Decision-aware findings and a decision-outcome study question (M4 and later)

- **Source**: Part A L14–24 (decision compliance, assumption validity, tradeoff evaluation, concept alignment as separate observer assignments), L43–53 (review proposals when a trigger is satisfied).
- **Gap**: M4 defined one narrow completion-claim observer and M3 deterministic rules; neither named decision-aware checks, and nothing asked whether a decision's expected benefit materialized.
- **Change (M4 item 5, defined but not scheduled, under the existing "measured gap" rule)**: two finding types for later specialists: **`decision_conflict`** (a patch or run contradicts a decision accepted and not superseded at the inspected revision's time; cites decision ID and revision, affected location, and expected consequence; deterministic where the decision carries a machine-checkable constraint, otherwise semantic) and **`revisit_proposal`** (a divergence-register or decision reconsideration trigger appears satisfied by retained evidence; cites the original decision or compromise, the evidence that its constraint changed, affected components, and estimated effort and benefit). Both are advisory and bound to the inspected revision like every Finding.
- **Change (M5 separable questions)**: a fourth question, "Did the expected benefit stated in a decision record materialize, measured as that record's Measurement field specifies, with its causality caveat?"
- **Trigger lifecycle**: lives in the P3 paragraph under the Divergence Register.
- **Milestone**: M4 and later; no observer is authorized or costed by this proposal.

---

## 5. Out of Scope for Tuan (parallel work only if ever wanted)

- **Enforcing ADR compliance on patches across Lugos.** The plan assigns execution, routing, and any intervention to Autowork and the harnesses (L18). Tuan may measure how often runs contradicted accepted decisions and whether expected benefits materialized (P8); it does not block patches.
- **A Lugos-wide ADR convention.** Today `torc`, `codelink`, `aeta`, `sulis`, and the Lugos root use three numbering styles, two templates, and no index or machine-readable status. A compliance observer across projects would first need a governance convention (machine-readable status and revision in decision-record front matter) under `config/agent-governance`. That is not Tuan's and has no value until decisions with evaluable consequences and P1 linkage data exist. Not started by this proposal.

---

## 6. Integration and Verification

- P1–P8 were applied as edits to the named sections of the kickoff plan: no new milestone, no renumbering of D1–D6, one new record row (Score), one new Accounting Rules bullet, one new Divergence Register row and paragraph, two new M3 items, one new M4 item, one new M5 question, and additions to the M0 inventory and task 4, the M1, M2, and M3 exit criteria, the retention rule, and the point-of-need decisions.
- Verification markers in the plan: `observed in effect`, `Estimate and score lineage`, `| **Score**`, `leakage status`, `lifecycle.md`, `tuan refresh`, `recency-weighting policy`, `long-lived compact evidence class`, `decision_conflict`, `revisit_proposal`, `A divergence is resolved only by`. `lugos-audit quick` must pass on the tuan checkout.
- The transcript file stays untracked until the M0 `docs/research/` move, then moves with its header (Section 2).
- The integrating commit message records the cross-review run ID and that all 11 findings were applied.

---

## 7. Cross-Review Findings and Dispositions

Run `20261030-060647-2-cross-review` (`codex-astra`, adversarial, 1800 s deadline, 11 findings received, 11 adjudicated `accepted`). Every finding was applied; none was rejected or deferred.

| ID | Severity | Finding (abridged) | Disposition |
| --- | --- | --- | --- |
| F1 | high | P6 treated a feature-transformation change as a scoring-rule change, breaking P2's recalculated-score vs revised-estimate distinction. | Evaluation-only rules rescore unchanged predictions; predictor-input changes require explicitly requested replays producing revised estimates; M2 fixture for a transformation fitted on later outcomes (P2, P6). |
| F2 | high | Score supersession had no identity or selection rules, risking double counting or replacing unrelated scores. | New Score record contract with deterministic IDs, full binding, series rule, retained unweighted series; M2 fixture for label correction, rule change, repeated run (P2). |
| F3 | medium | Leakage status had no defined values, evidence, or unknown behavior. | Three statuses with evidence rules; `unknown` never upgraded; four replay fixtures; revised estimates excluded from prospective accuracy regardless of status (P2). |
| F4 | high | A projection position alone cannot identify pending scoring work. | Durable evaluation-coverage records independent of projection progress; retry-safe identities; commit-after-write; four refresh tests (P6). |
| F5 | medium | Durable no-op receipt contradicted the byte-identical ledger requirement. | Receipts are operational records outside the evidence ledger; receipt-only changes are never pending work; no-op test covers both (P6). |
| F6 | medium | Weighted reporting required without a selected policy; group-normalized weights let one recent lineage carry the full recent share. | Opt-in until a recorded decision; minimum recent support with suppression; empty-group fallback; the full-share behavior is disclosed as intended (P5). |
| F7 | high | Weighted scores lacked temporal and membership inputs to be reproducible or to separate current reassessment from historical-as-of evaluation. | Policy and evaluation identity fix window basis, as-of time, population, contribution unit, label-selection rule; late corrections create a new as-of evaluation; four tests (P5, Score record). |
| F8 | medium | Decision and configuration identity semantics unresolved; producer feasibility unverified on either platform. | M0 inventory maps fields per platform with fixtures; M0 resolves namespace, revision, applicability, status evidence, scope, provenance, ordering; feasibility stated as unverified; `unknown` valid (P1). |
| F9 | medium | Mutable Status line conflicted with immutable accepted records. | Immutable content; lifecycle entries in `docs/decisions/lifecycle.md` and successor links; applicability evaluated at the run or inspected revision's time (P4, P1). |
| F10 | medium | P7 had no executable check that mixed retention keeps replay claims honest or that deletion reaches derived records. | M1 retention fixture and cross-class deletion exercise; class assignment, expiry clock, dependency behavior in the pre-import decision (P7). |
| F11 | medium | Integrating all eight unconditionally put optional maintenance and weighting into M2 before their policy and cost questions were resolved. | P5 and P6 moved to M3 with cost measured there; P3 merged with P8's trigger lifecycle; first-release scope unchanged (Section 1). |

# Kickoff plan: Fable cross-review

Date: 2026-10-31. Scope: review and correct the existing kickoff plan; no Tuan implementation, collection, or experiment is authorized.

## Round 1

Autowork run `20261031-080452-cross-review` succeeded using `claude-fable`, model `fable`, effort `high`, adversarial mode. The reviewed plan was at `fb4de8f`; packet manifest SHA-256: `8d8f8a00a8ec0e7b5946cc4a8c1f4dd4b46d263ed06e55a5e56ce1b7bb5504c9`.

Fable returned 20 findings: 2 high, 10 medium, 7 low, and 1 nit. It marked the first twelve must-fix and the remainder optional. Autowork's preliminary acceptance is not the root's adjudication. All twenty identify an in-scope consistency or acceptance concern, classified here as required by the existing measurement/privacy contract; several proposed remedies were corrected rather than adopted literally.

| Finding | Root disposition and correction |
| --- | --- |
| F-01: scoring retries | Defined the score binding; unchanged evaluation reuses the score. Label changes supersede within a series; rule changes create a separate series. |
| F-02: checkpoint dependency | Added a manual acquisition checkpoint path for M0–M2. First acquisition does not imply task start; M3 keeps automatic event triggers. |
| F-03: deletion identifiers | Separated non-sensitive completion receipts from protected refusal records. Stated the limits of matching unknown subject identities. |
| F-04: cross-ledger ordering | Accepted the gap, rejected receiver arrival order as proof of original issuance. Require origin evidence and a trustworthy causal ordering link; otherwise imported verification stays unknown. |
| F-05: redaction changes | Accepted policy versioning, rejected automatic supersession merely because the version changed. Require a verified transformation or sanitized rejection. |
| F-06: refresh after deletion | Added deletion/expiry dependency invalidation and classified coverage and diagnostic receipts for retention and deletion. |
| F-07: retention defaults | Made 90 days the floor and 180 days D4's proposal throughout. |
| F-08: topology decision | Marked storage topology as the D3 proposal; a different choice must update affected contracts and cases. |
| F-09: M1/M2 dependency | M1 uses labeled forecast/score fixtures and feature snapshots; M2 verifies predictor/evaluator behavior. |
| F-10: decision lifecycle | Named point-of-need dependencies, including D4 before real import. Synthetic work can use labeled assumptions; existing authorization remains valid. |
| F-11: producer identity | Allowed explicit unknowns with coverage and stable source identities; collector identity never fills a missing producer identity. |
| F-12: optional compact class | Made compact retention conditional; coverage-loss checks apply with or without it. |
| F-13: cutoff authority | Aligned the risk summary to ledger identity and ingestion position. |
| F-14: rule cases | Aligned M3 exits to all five required case types for checkpoints and rules. |
| F-15: weighting precedence | Insufficient recent support, including zero, suppresses weighting. Only an empty older group with sufficient recent support falls back. |
| F-16: no-op verification | Compare canonical logical contents rather than SQLite file bytes. |
| F-17: example | Corrected the forecast-error sign and stated target scope and the example's M4 observer cost. |
| F-18: terminology | Kept range method versioned, used label version consistently, and preserved assessed code/spec revision as a different identity. |
| F-19: writer ownership | Defined interprocess writer serialization and contention/interruption checks. |
| F-20: withdrawal timing | Clarified effects on future training and affected scores without rewriting earlier snapshots or leakage classifications; deletion/expiry can still remove evidence. |

Round 1 outcome: revise. A focused second round checks these corrections and their interactions; it is not another independent review of unchanged sections.

## Round 2

Autowork run `20261031-081630-cross-review` succeeded with the same Fable route and effort. It reviewed the corrected working-tree plan (file SHA-256 `3fec6d0de3d87d73a5726a8e8034e2e5bd2d8213c73c9c972ff481d6c62df342`); packet manifest SHA-256: `fe190b486456a303e55a5ef34dc9f57cc4352c29f7e7a4929815a7bb5cfa656a`.

Fable returned 14 findings: 1 high, 7 medium, 5 low, and 1 nit. Root disposition: the contract gaps and wording ambiguities below are required by the existing measurement/privacy contract. Literal remedies that would weaken evidence or deletion guarantees were rejected as described.

| Finding | Root disposition and correction |
| --- | --- |
| R2-01: delayed acceptance | Prospective eligibility now requires an open target before its source terminal event, separately from later acceptance/label receipt. A last usage record alone is not a terminal event: an open task may legitimately consume no further tokens. |
| R2-02: score anchors | Derive identity from policy-selected inputs/window membership before scoring; invocation time is receipt provenance. |
| R2-03: protected recovery authority | Added a protected maintenance stream to the complete recovery set, a first recovery pass, and scoped origin authority over collected copies. Real refusal keys stay out of fixtures. |
| R2-04: expiry versus deletion | Expiry invalidates dependencies selectively; sufficient compact evidence survives with explicit replay limits. Privacy deletion still reaches all applicable classes. |
| R2-05: restored writers | Historical receipt identity survives restore, but each restored writable database gets a fresh writer identity linked to its recovered boundary. |
| R2-06: redaction representation | Distinguished stable source event identity from normalized representation identity; one applicable representation contributes to accounting. Representation cleanup does not revoke a surviving source event. |
| R2-07: forecast/checkpoint identity | Defined deterministic bindings and reuse of unchanged acquisition checkpoints; derived writes do not create fresh checkpoints. |
| R2-08: retained history | D6 uses the actual retained-evidence horizon. Declining compact retention can leave promotion unavailable; no silent relaxation or retention extension. |
| R2-09: erased refusal keys | Rejected admitting all later-position records: they could reintroduce the deleted subject. Defined namespace-level restriction until coverage or authority is re-established. |
| R2-10: unknown producer | Distinguished configured source authority from observed producer host/PID. Unscoped input is quarantined; missing subject mapping remains an explicit matching limit. |
| R2-11: issuance assessment | Added separate issuance-verification assessments. Same-origin evidence must establish ordering against the terminal event, not just delayed label receipt; cross-origin proof remains explicit. |
| R2-12: expiry signal | Named the M1 storage-library expiry operation and atomic positioned maintenance receipt; no premature CLI verb. |
| R2-13: example label | Used captured-checkpoint wording and made percentage denominators explicit. |
| R2-14: M0 fixture | Clarified authorized manual sampling/redaction outside the ledger and the existing decision dependencies on real import. |

Round 2 outcome: revise. A third round is limited to these corrected contracts and their interactions.

## Round 3

Autowork run `20261031-082546-cross-review` succeeded with the same Fable route and effort. The reviewed plan's file SHA-256 was `bb79a4b1b6b6d6fdadb0c27669aecd380cbce8e8c1e482bff2d57edd4896a330`; packet manifest SHA-256: `8a96b9271d4d9cfeed63ba067619d3c4a07ed2fbfcd64796c6bd7c069f7d44f7`.

Fable returned five findings: four medium and one low. Root disposition: each clarification below is required by the existing measurement/privacy contract. The corrections are local contract clarifications; they do not introduce another milestone, service, or implementation task.

| Finding | Root disposition and correction |
| --- | --- |
| R3-F1: issuance reassessment | Narrowed the label-change rule. New ordering/lifecycle evidence can append an upgrade or downgrade; current populations follow the latest applicable assessment, preserving original claims and historical scores. Added both late invalidation and later verification cases. |
| R3-F2: compact eligibility | Included the minimum supporting eligibility metadata in the optional compact-class decision. Dangling references do not establish eligibility; unsupported retained scores stay out of promotion. |
| R3-F3: evaluation boundary | Froze the receiving evidence boundary and window-reference time separately. This excludes late records from historical populations while allowing time windows to advance when no new event arrives. The suggested use of boundary ingestion time alone would freeze idle windows, so it was not adopted. |
| R3-F4: re-acquisition | Permitted evidenced authoritative re-acquisition as a second migration path. A source dedup key alone does not prove unchanged content, and policy changes cannot restore deleted facts. |
| R3-F5: training cutoff | Made the existing cutoff rule explicit for every forecast kind and fitted transformation. Prospective issuance cannot use later training evidence through a reused checkpoint; the suggested expansion to a later effective cutoff was rejected. |

Round 3 outcome: five clarifications applied and checked locally. No fourth round was run. This is not a zero-finding Fable verdict; the root closed the bounded review after verifying the final corrections.

## Verification

Documentation checks passed for the unchanged milestone heading structure, unchanged M4/M5 and D1–D6 proposal sections, corrected terminology, fenced blocks, the final contract guards, and all 39 finding dispositions. `git diff --check` passed. The final plan's file SHA-256 is `a89b6957d6c1ea2f4b39ee1673c45e3430833dee42524c3800f80d7a9c706f22`.

These are documentation checks, not execution of the future acceptance tests. No Tuan implementation, source collection, model promotion, or experiment was performed. The final five corrections received root verification, not a fourth Fable verdict.

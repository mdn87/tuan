Yes. I found relevant research, and an unusually direct example in Torc’s own documentation. The failure pattern is treating “not enough evidence for the strongest claim” as “nothing useful can be measured or said.”

Torc already explicitly corrects that behavior. Its October 21 measurement plan says: “Saying there is no token-usage evidence is inaccurate.” It also tells future summaries to lead with measured results rather than substitute a blanket “no evidence” statement. Measurement plan

There are actual numbers. In the diagnostic’s one workload where both portable approaches passed, Torc reported 834,771 input tokens versus 578,622 for the plain handoff: 44.27% more, not fewer. That’s a quantified result for that comparison, even though it doesn’t settle whether Torc helps across your normal work. The diagnostic also records elapsed time, output and cached-input counters. Recorded results

A likely source of the repeated disclaimer: the current README still says token savings and live model trials remain unmeasured in its offline-milestone summary, while the newer plan documents real CLI measurements. An agent reading the overview and stopping there could recycle the older limitation. That’s a plausible explanation from the documents, not a verified reconstruction of each agent’s reasoning.

The research connection:
- An EMNLP paper calls a related behavior “abstention inflation”: prompt structure can make models decline answers they are capable of giving. Its experiments concern question-answering, not Torc, but show that abstention isn’t always a faithful signal of inability. Paper
- Google researchers found models poor at faithfully expressing uncertainty, including excessive as well as insufficient hedging. More caveats do not automatically mean better calibration. Research
For your projects, agents need to separate four questions:
1. What was directly counted? Report it.
2. What difference was observed against a baseline? Calculate it and name that baseline.
3. What can reasonably be estimated? Give assumptions and a range when supported.
4. What remains unestablished? Name that specific gap, such as broader effectiveness or actual subscription-credit savings.
The useful instruction is: “Do not turn missing proof of general savings into a refusal to quantify observed effects. Report the strongest supported result first, separate measurements from estimates, state the remaining limitation once, and identify the smallest next measurement that would resolve it.”

So I wouldn’t ask agents to be less honest. I’d ask them to stop confusing honesty with repeatedly withholding the useful part of the answer. I verified this document mismatch and the measurements; I haven’t audited the particular sessions you’re referring to.

----------------------

Here’s a kickoff prompt you can give the Torc session:

Start a new Torc subproject for context-impact measurement and usage forecasting.

I want to estimate how much token usage changes because of the context Torc provides: its selection, compression, continuity, retrieval, and preservation of useful information. Use our existing measurement work as a starting point for a new plan, potentially including a purpose-built measurement and reporting system.

The goal is useful engineering evidence and progressively better estimates. Don’t assume Torc saves tokens, but don’t treat the absence of definitive proof as a reason to stop quantifying what we can observe or reasonably estimate.

My brother suggested keeping a database of past requests, recording starting estimates and final counts, comparing projected difficulty with actual effort, then using that history to forecast usage and improve the design. Explore how that could fit what we already have. Treat “self-healing” as a possible later capability that needs a defined feedback loop and safeguards.

1. Establish what already exists.

Read the current Torc instructions and relevant implementation, especially:
- docs/plans/2026-10-21-measured-efficiency-plan.md
- docs/plans/2026-10-21-efficiency-execution.md
- experiments/session-continuation-001/diagnostic-results.md
- Existing continuation accounting, analysis, context projection, and reporting components.
Identify relevant capabilities in connected projects such as Autowork and the current Torc trial. Verify their actual state rather than assuming plans are implemented. Produce a reuse / extend / build inventory with source paths, ownership, and missing connections. Look for stale or conflicting documentation that could cause agents to repeat inaccurate limitations.

Preserve the original study and results. The diagnostic’s passing comparison reported 44.27% more input tokens for Torc; retain that finding without treating it as a verdict on every context strategy or workload.

2. Separate the questions we need to answer.
- What tokens, work, time, and outcomes were observed?
- What usage should we forecast for a new request?
- What difference can reasonably be attributed to Torc’s context strategy?
- Under what circumstances does that strategy help, hurt, or make little difference?
Distinguish smaller initial context from lower total task usage. Better context might cost more initially while preventing repeated investigation, mistakes, retries, or lost requirements. Measure both possibilities. Forecast accuracy alone does not establish that Torc caused a benefit.

3. Design the request-history and accounting foundation.

Propose a minimal data model linking requests, task families, attempts, calls, context versions, estimates, outcomes, and source evidence. Record starting estimates before execution, with timestamps and estimator versions. Keep them immutable alongside actual results so we can evaluate prediction errors without hindsight leakage.

Consider features such as task type, repository, difficulty estimate, model, settings, starting context size, context strategy, and prior relevant history. Evaluate whether “difficulty” should be a single score or several features; don’t assume difficulty scores or usage across models can simply be added.

Separate reported input, output, cache reads/writes, estimated counts, missing data, and billing information. Avoid double counting cumulative counters or parent/child calls. Include failed and abandoned work, retries, context preparation, summarization, retrieval, and maintenance overhead. Explain how reusable preparation costs are allocated without charging them twice.

Keep sensitive request content out of the database unless necessary. Propose retention, redaction, provenance, and access boundaries.

4. Develop a practical attribution plan.

Define specific context interventions and fair baselines. Compare equivalent tasks and outcomes, controlling or accounting for model, tools, settings, cache behavior, task difficulty, and execution conditions.

Propose a progression from inexpensive measurements and instrumented real use to matched comparisons or randomized trials where feasible. For observational estimates, state the assumptions and likely confounders. Include task success, quality, latency, and user effort so cheap failures cannot appear as efficiency wins.

Do not silently inherit every gate from the old experiment or waive its requirements. Explain which requirements belong to that frozen study, which belong in this new subproject, and why each proposed gate is proportional to the claim being made.

5. Add forecasting and a controlled improvement loop.

Start with simple forecasting baselines before sophisticated models. Define chronological validation, prediction ranges, calibration, drift detection, and what enough data means for each decision. Explain how historical requests will improve estimates without using final outcomes as pre-execution features.

Progress from reporting to recommendations, then human-approved context-policy experiments. Any eventual automatic adjustment needs bounded authority, quality checks, an audit trail, rollback, and separate evaluation data. Avoid optimizing token counts at the expense of useful work.

6. Make reports useful even before definitive results exist.

Lead with measured values and supported estimates. Clearly label observed, estimated, experimental, and unknown quantities. Name the baseline and reporting period. State each material limitation once, then identify the smallest next measurement that would reduce it.

Show an example report that answers: what happened, what we expected, the estimated context-related difference, confidence and assumptions, overhead, outcome quality, and what to change or test next. Use clearly labeled illustrative numbers if necessary, never fabricated project results.

Deliver a new planning package inside Torc:
- Existing-capability inventory and gaps.
- Clear measurement definitions and proposed data model.
- Architecture and ownership boundaries.
- Attribution and forecasting plans.
- Example report and phased roadmap.
- The smallest useful first implementation milestone with acceptance criteria.
- Open decisions requiring my input.
Keep an ideal conceptual design alongside the practical implementation plan, with a short divergence register explaining compromises and when to revisit them. Explain the plan in plain English as well as technical detail.

This assignment is discovery and planning. You may draft the new planning documents, but do not implement the system, launch experiments or background collectors, install services, change credentials, spend money, or overwrite the original study. Finish with your recommended first milestone and the decisions needed to proceed.










this likely should just be a new project entirely actually, something we can use parallel with our current memory system... here's more shaping... lets reshape the prompt and spitball some on-theme for lugos name ideas you know 3 to 5 letters long maybe 6 tied to celtic mythology or storytelling or history...

Yessir. You can try different predictive models to see what fits your flow best. Or have them compete for accuracy if you're unsure, just run them in parallel.

Usually worth it. Proactively tracking things ends up being more efficient and you can shape the data over time. Keep the logs though, 90 days +. It's cheap and let's you reshape the data structure as you refine the design. And log literally everything with timestamps and process identification.











Yes, that makes more sense as a separate project alongside the memory system, with Torc as one integration rather than its owner. I’ll reshape the prompt around a durable event history, competing prediction models, and context-impact reporting, then pull together a few short Celtic names.


----------------------------

Names worth considering
- Tuan · 4 letters. Tuán mac Cairell preserves and recounts history across successive transformations. My favorite for a system whose evidence survives changing agents and prediction models. UCC
- Fintan · 6 letters. A legendary figure associated with Ireland’s remembered history. A strong alternative if you want a character name rather than a functional word. Ulster University study
- Ogham · 5 letters. The early Irish writing system: a fitting historical reference for durable records. It sounds close to your existing Ogmi/Ogmios names, though. National Museum of Ireland
- Lorg · 4 letters. Irish for a track or trace. Less mythological, more directly tied to tracing what happened. Dictionary
- Tomhas · 6 letters. Irish for measure, with senses including estimation and guessing. Particularly apt for this project’s purpose. Dictionary
Tuan for the story; Lorg for the function. These are naming candidates, not checked repository/package/trademark availability. I’ve left the name open in the prompt.

Revised kickoff prompt

Plan a new, standalone project in the Lugos ecosystem for agent-work measurement, context-impact estimation, and usage forecasting. Use PROJECT_NAME as a placeholder until I choose a name.

This project should operate alongside our current memory system. Torc is an important integration and source of prior research, but this is not a Torc subproject. The design should work across agents, models, repositories, and context strategies without becoming another competing memory store or taking over execution orchestration.

The central idea is to preserve a trustworthy history of what we expected, what actually happened, and what context was supplied, then use that history to improve estimates and evaluate design changes.

My brother suggested keeping a database of past requests, recording starting estimates and final counts, comparing projected difficulty with actual effort, and eventually using predictive analytics to forecast usage and optimize the system. He also suggested running competing predictive models in parallel, keeping logs for 90 days or longer, and recording comprehensive timestamped events with process identification so we can reshape our analyses later.

Use those ideas as design input, not assumptions that every detail is already correct or that self-improvement happens automatically.

1. Inspect what we already have.

Inventory relevant components in Torc, the current memory system, Autowork, and other applicable Lugos projects. Identify existing request identifiers, usage capture, event logs, context construction, persistence, evaluation, reporting, and unfinished integrations.

Start with Torc’s:
- docs/plans/2026-10-21-measured-efficiency-plan.md
- docs/plans/2026-10-21-efficiency-execution.md
- experiments/session-continuation-001/diagnostic-results.md
Verify current source and deployment evidence separately. Classify components as reuse, extend, or build, with source references and ownership. Preserve existing studies and results. Treat their requirements as study-specific unless there is a reason to adopt them here; do not silently waive old gates or impose all of them on this new project.

2. Define the project’s responsibilities.

Design three connected capabilities:
- Observation: what context, usage, work, and outcomes occurred?
- Forecasting: what usage and effort should we expect before a request runs?
- Attribution: how much difference can reasonably be attributed to context selection, compression, retrieval, continuity, or memory use?
Keep these distinct. Accurate prediction is not proof of causation. Smaller starting context is not necessarily lower total effort. More initial context may prevent repeated investigation, mistakes, or lost requirements, or may simply add overhead.

Describe the integration boundary with the memory system: consume relevant versioned metadata and expose reports or recommendations without silently modifying memories or duplicating their contents.

3. Preserve evidence while allowing the analysis to evolve.

Propose a durable event history and database with at least 90 days of useful operational records, configurable longer retention, and rebuildable analytical views. Compare simple storage options before proposing infrastructure.

Keep historical observations intact, version schemas and transformations, and make it possible to replay retained events into a revised data model. Append-only should prevent silent rewriting, not prevent authorized privacy-related deletion.

Record event time and ingestion time, request/task/run identifiers, parent-child relationships, attempt identifiers, provider/model/settings, context-policy versions, and process identity. Include host and process-start identity because a PID alone can be reused. Address clock skew, duplicate events, late arrivals, retries, restarts, and incomplete records.

Interpret “log everything” as comprehensive relevant operational evidence. Do not persist secrets, credentials, hidden reasoning, or unrestricted prompt/response/memory contents. Define redaction before persistence, access controls, payload opt-ins, retention classes, deletion, and storage limits. Measure collection overhead rather than assuming logging is free.

4. Connect starting estimates to actual outcomes.

Store forecasts before execution with their timestamp, as-of data cutoff, estimator version, feature snapshot, predicted range, and assumptions. Preserve forecast revisions alongside the original instead of replacing it with hindsight.

Consider task type, repository, model, context strategy, starting context size, difficulty features, and relevant prior history. Evaluate whether difficulty needs multiple dimensions rather than one additive score.

Track reported input/output/cache categories separately, with provider-specific definitions. Distinguish tokens, estimated cost, actual billing, subscription allowances, elapsed time, and human effort. Include failed and abandoned tasks, retries, child calls, context preparation, retrieval, summarization, and maintenance. Prevent double counting and explain how reusable preparation costs are allocated.

Track successful completion and quality alongside usage. A cheap failure is not a savings success.

5. Let prediction models compete fairly.

Start with a simple baseline and one challenger before adding more sophistication. Support parallel shadow predictions using the same information available at prediction time.

Compare accuracy, bias, interval coverage, performance by task type, inference cost, latency, and maintenance burden. Use chronological evaluation and held-out periods. Prevent future information leaking into features or evaluation outcomes leaking into training.

Define how model selection, ensembles, retraining, and drift detection would work. Do not assume one global winner is best for every workload. Recommend evidence-based data-sufficiency criteria rather than an arbitrary waiting period.

6. Estimate context-related impact at appropriate levels of confidence.

Define concrete context interventions, comparison baselines, and the effect we want to estimate. Propose a progression from descriptive measurements and clearly labeled observational estimates to matched or randomized comparisons where practical.

Account for model changes, task difficulty, tools, cache behavior, execution conditions, and outcome quality. Include preparation and reporting overhead in net-impact estimates. Report positive, negative, and inconclusive results without changing the rules to favor Torc or another strategy.

Lead reports with the strongest supported measurement or estimate. Label assumptions, coverage, and uncertainty. State each material limitation once, then identify the smallest next measurement that would improve the answer. Do not substitute “cannot prove savings” for reporting the useful evidence we already have.

7. Design a gradual improvement loop.

Begin with observation, forecasts, and recommendations. Then propose human-approved context-policy experiments. Any future automatic adjustment needs bounded authority, quality safeguards, versioned policies, auditability, rollback, and evaluation that does not reward manipulating its own metrics.

Explain what “self-healing” would actually mean here: what detects a problem, what can change, how improvement is verified, and when the system must stop or ask. Keep automatic memory changes, context tuning, and agent routing disabled in the initial design.

Deliverables

Produce a standalone project seed and planning package containing:
- A plain-English concept and ownership boundaries.
- An evidence-backed reuse/extend/build inventory.
- Proposed event schema, storage and retention design, and integration contracts.
- Measurement, attribution, and forecasting plans.
- Example reports using clearly labeled illustrative data where needed.
- A phased roadmap, risks, and decisions requiring my input.
- An ideal conceptual plan alongside the practical plan, with a divergence register.
Recommend the smallest useful first milestone: one instrumented workflow, replayable records, a baseline and challenger forecast, and an understandable report. Decide what can be built on existing infrastructure before adding services.

This is a discovery-and-planning assignment. You may draft the seed and planning files in the designated workspace. Do not create or publish a remote repository, install collectors or services, change credentials, launch paid/model experiments, alter the memory system, or enable autonomous optimization. Finish with the proposed first milestone and the specific approval needed to implement it.
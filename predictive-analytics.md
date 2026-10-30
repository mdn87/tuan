Predictive analytics for coding means using the history and current state of a task to estimate what happens next: how much work remains, how many tokens it might consume, whether it is heading toward a failure, or whether a change in approach could help.

The parallel-model idea is useful because you can have several observers evaluate the same work while the coding agent continues. Some observers can compete to make the best forecast. Others can specialize in spotting different problems.

For the separate project you’ve been discussing alongside Torc, I would build around a shared event history, independently evaluated observers, and one place that decides whether their findings should affect the task.
1. “Different models watching” covers two different jobs
Numerical forecasting
These models take structured measurements and estimate an outcome.

For example:
Given the task category, repository, model route, context size, and failures so far, how many tokens are likely to remain?

A useful starting lineup would be:
Approach	How it works	Why include it
Historical baseline	Uses the median and range from similar previous tasks	Gives more complicated models something inexpensive to beat
Regression model	Learns relationships between task characteristics and actual effort	Makes it relatively easy to inspect which measurements influence an estimate
Tree-based model	Learns combinations of conditions that affect outcomes	Can capture interactions such as large context combined with repeated test failures
Online learner	Updates as new outcomes arrive	Lets you investigate whether adapting to recent work improves forecasts


These can be ordinary statistical models running locally. They do not require an LLM request every time a measurement changes.

I would also distinguish the initial total-work estimate from the remaining-work estimate during execution. They answer different questions and have access to different evidence.
Semantic review
An LLM observer can interpret things that counters alone cannot explain:
- Does the implementation still satisfy the original request?
- Did the agent misunderstand a requirement?
- Does a passing test actually exercise the behavior being changed?
- Is the agent repeating an approach whose underlying assumption has already failed?
- Has a temporary implementation compromise quietly become a permanent design decision?
This is particularly relevant to your idea of maintaining an ideal concept alongside the implementation plan. A reviewer could compare a patch with both documents and propose a divergence-register entry, including the affected requirement and supporting evidence.

Parallel specialist review is already an implemented pattern. Anthropic’s Code Review documentation describes multiple agents examining different issue classes concurrently, followed by verification, deduplication, and prioritization of their findings. That demonstrates the architecture, although it does not establish that it would save tokens in your workflow. Claude Help Center

I would use statistical models for repeated numerical predictions and LLMs for bounded interpretation at meaningful checkpoints.
2. Three ways to organize the parallel work
Competing predictors: several models answer the same question
Suppose three models receive the same task checkpoint:
Estimate remaining token usage until this task meets its acceptance criteria.

They independently produce forecasts. You record all three, then compare them with the eventual outcome.

This is commonly organized as a champion/challenger system:
- The current champion supplies the forecast you display.
- Challengers make their own forecasts in the background.
- Their performance is compared on subsequent tasks.
- A challenger replaces the champion when the evidence justifies it.
Initially, challengers should operate in shadow mode, meaning their predictions are recorded without changing the task. AWS documents this general pattern as copying live inputs to a candidate model while the existing model supplies the operational response. You can apply that pattern without adopting AWS’s infrastructure. Amazon SageMaker AI

This directly matches your brother’s suggestion to let different predictive approaches compete.
Specialist observers: each watches a different aspect
Here, agreement is not the goal. Each observer should contribute useful information the others would miss.

A proposed lineup could look like this:
Observer	Question it answers	When I would run it
Usage forecaster	How much time or usage remains?	Task start and meaningful checkpoints
Loop detector	Are the same actions or failures repeating without new evidence?	As relevant events arrive
Requirements reviewer	Is the work still addressing the requested outcome?	After planning and substantial changes
Verification reviewer	Does the available evidence support the completion claim?	At verification and completion
Context analyst	Is the agent repeatedly missing, rediscovering, or receiving irrelevant context?	At retrieval, compaction, and handoff checkpoints


The loop detector might begin as straightforward rules. The requirements reviewer would probably benefit from an LLM.

Their scorecard should measure additional confirmed findings, false alarms, useful warning time, and cost. Counting how often they agree would tell you relatively little.
An ensemble: combine several predictions
An ensemble combines model outputs into one result, perhaps through a weighted average or a learned combination.

I would introduce this after measuring the individual predictors. You need to know whether combining them improves performance and whether their errors overlap.

Different model providers do not guarantee independent judgment. An Apple study of LLM evaluation panels found substantial correlated errors across model families. Its experiments concerned language-inference evaluation, so it does not provide a numerical estimate for coding reviewers, but it supports testing independence instead of assuming it. Apple Machine Learning Research

For code review, a reproducible finding should carry more weight than a majority vote.
3. How the observers should actually run
The mechanism can be fairly concrete.
Capture a checkpoint
A useful checkpoint might occur when:
- The initial plan is ready.
- A coherent patch has been produced.
- A test failure repeats.
- The task scope changes.
- A handoff or context compaction occurs.
- The coding agent claims completion.
At that point, create a versioned snapshot containing the task definition, relevant code revision, recent events, available test results, and accumulated usage.
Dispatch independent observations
Each observer gets the evidence appropriate to its job.

A numerical predictor might receive a small feature record. A semantic reviewer might receive the task, relevant diff, selected source files, and recent failures.

For competing models, hold the evidence cutoff constant. A predictor that has seen three failed test runs has an advantage over one predicting before execution begins.

Record each model’s original result before exposing it to other models’ opinions. That preserves your ability to evaluate its independent contribution.
Record forecasts and findings separately
A forecast needs fields such as:
- Target being predicted.
- Prediction time and checkpoint.
- Model and feature versions.
- Estimate and uncertainty range.
- Cost of producing the estimate.
A review finding needs somewhat different information:
- Specific claim.
- Relevant requirement or code location.
- Evidence.
- Expected consequence.
- Suggested verification.
- Revision to which the finding applies.
This distinction matters because “likely to exceed its budget” and “this implementation drops an existing behavior” need different forms of validation.
Verify and surface useful findings
I would have a single advisory stage deduplicate findings, check whether they are still current, and decide what to surface.

An observer that finishes after the working code changes should retain its result against the revision it inspected. A finding that concerns changed code needs revalidation.

If reviewers execute tests, give them isolated checkouts or equivalent environments. Concurrent observers that alter the same working directory can interfere with the task they are measuring.

OpenAI’s code-verification research reports benefits from repository access and execution tools, and emphasizes the cost of noisy findings to developers. That supports requiring evidence and verification rather than repeatedly interrupting with vague concerns. alignment.openai.com

Continuous tracking can support occasional model calls. You can capture every relevant event while invoking expensive reviewers only when something meaningful changes.
4. Proactive tracking starts with a useful history
Your brother’s suggestion to retain logs for 90 days or longer gives you room to revisit the data as the design evolves. I would make those logs structured and append-only, with derived summaries that can be regenerated.

The most useful categories would be:
Category	What to capture	What it lets you investigate
Identity and lineage	Task, session, run, agent, process instance, parent operation	Which work belongs together across agents and resumptions
Execution state	Start, pause, resume, tool calls, failures, retries, completion	Where time and effort accumulated
Code and intent	Repository revision, patch snapshot, specification and plan revisions	What each model actually inspected
Context	Retrieved references, injected context size, compactions, handoff revisions	Whether context delivery relates to repeated work
Usage	Provider-reported tokens, cache categories, model route, latency, observer usage	What the complete workflow consumed
Outcomes	Acceptance checks, failures, abandonment, later rework, human intervention	Whether the work produced a useful result


A timestamp and operating-system process ID are insufficient for tracing work across sessions and machines. I would also assign stable task/run identities and propagate operation relationships.

OpenTelemetry provides the relevant foundation: trace and span identifiers can connect logs and operations across process boundaries. Its “context propagation” refers to tracing metadata, which is a separate concept from the prompt context Torc manages. OpenTelemetry

A few accounting details deserve attention:

Keep elapsed time separate from active work. Waiting for a user, provider queue, or long build should be identifiable.

Keep observer usage separate from coding usage. Parallel execution can reduce waiting while increasing total consumption.

Preserve provider usage details. Cached-token counts can be subsets of input totals. Adding every reported number together can double-count usage. OpenTelemetry’s current GenAI conventions explicitly describe that relationship. GitHub

Represent missing measurements as unknown. A provider that does not expose a usage field has not necessarily used zero.

For raw content, I would retain useful evidence references and redact secrets before persistence. OpenTelemetry’s guidance supports collecting data that serves an observability purpose and filtering sensitive attributes. OpenTelemetry
The patterns worth watching
Some promising signals are sequences rather than individual measurements:
- Repeatedly reading the same unchanged files after context resets.
- Repeating a test failure without changing the relevant assumption.
- Reverting and recreating substantially similar patches.
- Expanding the diff while acceptance criteria remain unresolved.
- Spending increasing effort recovering information already available earlier.
- Declaring completion while an explicit requirement has no supporting verification.
Each is a reason to inspect the task. None should automatically count as failure. Repeated reading, for example, can be appropriate verification.

That is where the numerical detector and semantic reviewer complement each other: one identifies an unusual pattern, and the other assesses its meaning.
5. How to find out which models are actually useful
The evaluation needs to be designed before you start selecting winners.
Save the prediction before the answer exists
A model should make its prediction, have that prediction recorded, and only later learn the outcome.

This is especially relevant because coding outcomes arrive at different times. Token totals might become available at task completion, while a regression could be discovered much later. River’s progressive-validation tools explicitly support delayed outcomes of this kind. River
Test on later, separate tasks
One task might generate hundreds of checkpoints. Those are related observations.

Randomly distributing those checkpoints between training and evaluation can let the model effectively recognize the same task on both sides. I would keep task lineages together and evaluate using later tasks; claims about cross-repository performance would need a separate repository holdout.

This applies the grouped and temporal evaluation principles documented by scikit-learn. scikit-learn 1.9.1 documentation
Score the thing each observer is supposed to do
For usage forecasts, measure error and the cost of underestimating a task.

For prediction ranges, measure both coverage and width. A range advertised as containing 90% of outcomes should be checked against held-out results. Merely generating lower and upper estimates does not establish calibration; scikit-learn’s quantile-regression example demonstrates that nominal and observed coverage can differ. scikit-learn 1.9.1 documentation

For warnings, measure confirmed findings, missed problems, false alarms, and whether the warning arrived early enough to help.

For the whole system, measure accepted results, rework, total consumption, and interruptions.

Keep failed and abandoned runs in the dataset. Their cost is part of the workflow. An unfinished run’s cost-so-far also needs to remain distinct from completed-task cost.
Watch the predictors for deterioration
A model upgrade, new harness, different repository, or changed task mix may make old relationships less useful.

You can monitor recent prediction errors and use drift detectors to flag changes. For example, River’s ADWIN compares recent portions of a data stream using an adaptive window. Such a signal should trigger investigation, with model and harness version markers helping explain what changed. River
6. Forecasting usage and proving Torc’s benefit need separate tests
This is the most consequential distinction for your original measurement idea.

Suppose, hypothetically:
A forecaster predicts 80,000 tokens. A task using Torc finishes in 55,000.

That difference contains both prediction error and any effect from Torc. You cannot directly attribute the entire 25,000-token difference to improved context.

A causal question requires a comparison between specified conditions:
How does task cost and quality change when eligible work receives context treatment A versus treatment B?

Randomized assignment, where practical, provides stronger evidence than comparing unrelated before-and-after work. Controlled experimentation is used in software precisely because properly designed randomization supports causal conclusions. Microsoft Research

For your study, I would define:
- The exact context treatments being compared.
- Consistent completion and quality criteria.
- How task difficulty, model route, and repository are handled.
- Which preparation, review, retry, and recovery costs count.
- Whether observer advice can influence the run.
That last item is easy to miss. Once a watcher changes the task’s direction, you are evaluating the watcher plus its intervention.

If it warns about a defect and the agent fixes it, the eventual absence of that defect does not make the warning a false alarm. Record what was shown, what action followed, and what evidence supported the finding.

You could eventually separate three questions:
1. Does improved context help?
2. Do observer interventions help?
3. Does the combination help enough to justify its total cost?
Forecasting models can help organize these experiments, but their estimates cannot independently establish the counterfactual.
7. What I would build first for your project
I would keep the initial implementation focused on a small, testable loop:
1. Capture a complete task history. Reuse the evidence, usage, and lineage sources already available in your ecosystem.
2. Establish a cheap baseline. Start with historical estimates and simple repetition/failure rules.
3. Add competing forecasts in shadow mode. Compare a small number of approaches using the same checkpoints.
4. Add one semantic reviewer at selected checkpoints. Give it a narrow job, such as detecting requirement drift or unsupported completion claims.
5. Measure additional value before expanding. Each extra observer should contribute useful findings or better forecasts at an acceptable cost.
6. Introduce interventions separately. Let the existing execution system act on selected findings, while recording those actions for evaluation.
This fits the boundaries you’ve described: Torc supplies continuity and context evidence, the separate analytics project produces forecasts and findings, and the execution/UI components consume them.

You can also replay historical checkpoints when evaluating new observers. MLflow supports evaluating saved traces repeatedly, which illustrates how you can reuse execution evidence without rerunning the entire coding task for every evaluation. Historical prediction tests still need a strict evidence cutoff so a model cannot see what happened later. MLflow AI Platform

A useful first acceptance test would be: take one completed coding task and reconstruct what each observer knew, what it predicted at each checkpoint, what actually happened, and what the observation itself cost. Once that record is dependable across tasks, you have a basis for choosing models and testing proactive interventions.
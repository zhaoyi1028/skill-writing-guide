# Validation and iteration

Contents: 1. Three kinds of validation · 2. Test cases · 3. Comparison baseline · 4. Trigger testing · 5. Failure attribution · 6. Maintenance

## 1. Three kinds of validation

These answer different questions. Keep them separate, or a failure can't be traced to a layer.

| Type | What it checks | Method |
|---|---|---|
| Structure | Metadata valid, references resolvable, dependencies available, scripts run | Static checks plus a trial run |
| Trigger | Selected when it should be; not selected when irrelevant or merely keyword-similar | Don't name the skill explicitly; include near-miss negatives |
| Effect | Real task completed, output correct, no overreach, cost reasonable | Invoke the skill explicitly, isolating the selection stage |

## 2. Test cases

Start with 3 to 5 realistic cases, at least:
- a typical task;
- an out-of-scope task (should not trigger);
- an edge case (missing input, dirty data, tool failure, partial completion).

For complex or high-risk skills, expand based on risk and recorded failures. Cases should sound like real users: concrete file names, context, casual phrasing, even typos. Don't write abstract prompts like "process the data".

Give each case an observable assertion of what "good" looks like, e.g. "the summary total matches source data", "when refund dates are missing, the output separates confirmed results from attributions that can't yet be confirmed". Don't force assertions on subjective outputs (writing style, design); leave those to human review.

## 3. Comparison baseline

Compare "with skill / without skill" (or "new / old" when improving an existing skill). Hold constant: model, tools, inputs, initial state.

Evaluate:
- task completion quality;
- key errors and omissions;
- unnecessary questions and actions;
- tool calls, retries, duration, and token cost.

Check what files were actually read, what tools were actually called, and what was actually produced, not whether the model claims it followed the steps. Do targeted regression for routine edits; widen the comparison for major workflow changes.

Single runs have variance. When a conclusion rests on one or two runs, state the sample size and don't claim "significantly better".

## 4. Trigger testing

Label each case's expectation before testing and record the reasoning. Don't decide after seeing results whether a case counts:
- Should use: formal and casual phrasings, including cases where the user doesn't name the skill or file type but clearly needs it.
- Should not use: mostly near-misses, i.e. requests that share keywords or concepts but belong to a different tool; plus a few obviously unrelated cases as a baseline check.
- May complete directly: simple tasks the model can do well without the skill. Keep these to detect unnecessary loading. If such a task actually needs a key rule from the skill, "simple" is not an excuse for a missed trigger.
- For tasks where several skills may cooperate, record which skills are expected and which should not load. Don't assume a single winner.

Scoring: recall is computed over "should use" cases only; false-trigger rate over "should not use" cases only. Report "may complete directly" cases separately (whether loaded, quality, cost); they count as neither missed nor false triggers. For small samples, report successes over total (e.g. 7/9), not just a percentage.

Too many false triggers: narrow the scope or add one sentence of boundary. On a missed trigger, check four things first: is the skill discoverable, does the invocation configuration allow it, is the description fully visible in the listing (it may be truncated), is a neighboring skill winning instead. Change the description only after confirming it is the cause. Change once, retest once.

## 5. Failure attribution

Decide which layer the problem lives in, then fix that location:

| Symptom | Check |
|---|---|
| Skill not selected | Discoverability, invocation config, description visibility, competition from neighboring skills; change the description only once confirmed |
| Selected but wrong path taken | Task differentiation and resource navigation |
| Operation failed | Tools, dependencies, scripts |
| Wrong result declared complete | Completion criteria and acceptance |
| Did more than asked | Scope, authorization, and overly broad requirements |

Fix the responsible location. Don't append a new prohibition to the end of the body after every failure; that breeds duplication, conflicts, and maintenance burden.

Read failure traces to find the step where the model drifted and fix that step's guidance. If the model independently wrote similar helper scripts across several cases, that is evidence for extracting a script; before doing so, confirm the logic is stable and that existing tools can't already handle it.

## 6. Maintenance

- Delete duplicated, irrelevant, or obsolete content.
- For content with no observed effect yet, judge against its scenario. Low-frequency but important failure handling (confirmation before irreversible operations, retry checks after timeouts) must not be removed just because normal cases didn't use it.
- When the model, tools, or business rules change, re-check whether old guidance is still necessary and accurate.

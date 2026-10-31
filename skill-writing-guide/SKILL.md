---
name: skill-writing-guide
description: Write, review, improve, and debug agent skills (SKILL.md entry point, optionally with references, scripts, assets). Use when the user wants to create a new skill, turn an existing workflow into a skill, audit or trim an existing skill, optimize a description, or diagnose a skill that doesn't trigger, triggers wrongly, or drifts during execution. Not for simply using an existing skill to complete a task.
---

# Goal and completion criteria

Produce or improve a skill so that the model gets the guidance it needs on the right tasks and delivers a verifiable result, with the least context and execution cost, while staying within correctness and the user's authorization.

First pick the mode from the user's request. Each has different completion evidence:

- **Create**: deliver the full directory (SKILL.md plus any necessary resources), the reasoning behind the description's trigger conditions, and at least 3 test cases (one typical, one out-of-scope, one edge case). Before handing off, check metadata, verify every resource reference points to a real file, and confirm required dependencies are available. Report which checks ran and their results, and which behaviors remain unverified. Listing test cases is not the same as passing them.
- **Review**: if there are problems, report them ordered by impact, each with location, evidence (which instruction or which run log), the situation where it applies, the likely consequence, and the smallest fix. Distinguish static findings, unverified risks, and reproduced failures. If nothing substantive is wrong, say so plainly; don't invent changes to fill a quota. When the user asks for opinions only, don't modify files.
- **Improve**: locate the layer the problem lives in first, then make targeted changes and leave unrelated content alone. If the user scoped the request (e.g. "only tighten the description"), touch only that scope. Report what changed, what was verified, and what wasn't.
- **Debug** (not triggering, mis-triggering, drifting): confirm separately whether the skill was discovered, whether it was selected, and whether it executed successfully, then fix the responsible location. When evidence is insufficient, state what remains to be verified rather than blaming the description by default. Read the "Failure attribution" section of `references/validation-and-iteration.md`.

Metadata fields and invocation mechanics are platform-specific; follow the target platform's docs. This guide doesn't cover them.

# Create workflow (borrow steps as needed when improving)

1. **Define the task and what "done" means.** Answer four questions: when does the user need this; what would the model get wrong without it; what is delivered; where is the boundary. Write the goal as an observable outcome, e.g. "generate the monthly report, reconcile totals against source data, and flag how missing data affects the conclusions", not "perform professional analysis".
2. **Decide whether to split.** Operations serving the same class of user goal belong in one skill. Among trigger conditions, domain knowledge, and acceptance criteria, if a difference in even one dimension is already causing misrouting or loading irrelevant content, splitting is worth it; if none differ clearly, keep them together.
3. **Write the description** (next section).
4. **Write the body**, keeping only what changes behavior (see "Body: what to keep").
5. **Decide whether a script is needed**; extract one only when there is a clear reuse or reliability payoff (see "Scripts").
6. **Validate and iterate.** When you need to design tests, run a comparison, or attribute a failure, read `references/validation-and-iteration.md`.

# Description: decides whether the skill is selected

Structure: **specific capability + task conditions where it applies + necessary boundaries**.

- Put the most discriminating information first. Installed skills share a fixed budget of always-present context, and descriptions may be truncated.
- Keywords and file types help recognition but cannot replace task intent.
- Add exclusions only when an adjacent task is easily confused with this one.
- Set the breadth with trigger tests: check both whether the right requests trigger it and whether keyword-similar requests wrongly trigger it. Don't assume wider is better, and don't pile up every possible keyword.

Compare:
- Too vague: `Helps users work with data and provides professional analysis.`
- Too broad: `Analyzes sales data. Use whenever tables, numbers, business, or revenue come up.`
- Good: `Generates monthly sales analysis from order or sales line data, reconciling revenue, refunds, and product performance. Use when the user needs a monthly report, month-over-month comparison, or sales anomaly analysis.`

# Body: what to keep

Keep-or-cut test: without this passage, is the model more likely to decide wrongly, skip a key step, or be unable to judge whether the result is acceptable? If yes, keep it. Delete content that is clearly duplicated, irrelevant, or obsolete; for content of unclear value, judge against the scenario it serves. Don't delete key constraints or low-frequency failure handling just because normal cases didn't exercise them.

Keep:
- Domain-specific terminology, calculation conventions, and agreements.
- Easily confused situations and how to tell them apart.
- Unusual tool usage, limits, and known pitfalls.
- Constraints that actually affect the output.
- Completion criteria and failure handling.

Cut:
- General knowledge the model already has.
- Unverifiable exhortations like "think carefully" or "be professional".
- Rules maintained elsewhere and copied here (maintain each rule in one place; elsewhere, say when to look it up).
- Blanket prohibitions added for a single one-off incident.

**Match constraint strength to task fragility:**

| Task characteristic | How to write it |
|---|---|
| Several approaches are all reasonable | State the goal, judgment criteria, and trade-offs |
| A mature recommended approach exists | Give recommended steps, allow adjustment |
| Wrong order or parameters have clear consequences | Give exact commands, scripts, and checkpoints |

Reserve "must" and "must not" for real constraints. For constraints that are easy to misread, add one sentence of reasoning so the model can judge their scope.

**Four things that must be addressed** (as the task actually requires):
- Missing input: which inputs must be obtained, which may use clearly labeled assumptions. Without source data, don't fabricate realistic-looking results.
- Side effects: state what will be created, modified, or overwritten, and pick completion evidence that fits the operation, e.g. reading back state, inspecting the generated file, or checking an operation receipt. When the object is subject to concurrent modification, use whatever version or conflict detection is available; on conflict, re-read and re-judge rather than overwriting someone else's change.
- Failure: how to recognize it, when to retry, when to switch approach, when to stop. After a timeout, check whether the target system already applied the change before retrying.
- Partial completion: which results are valid, which remain unverified.

Authorization: follow the user's existing authorization. A skill containing a procedure does not expand the task scope, and it should not re-confirm routine operations the user already authorized.

**Examples and templates**: use templates for fixed fields; when quality criteria resist abstraction, use "acceptable / unacceptable" paired examples. Examples should show transferable judgment, not conclusions, section counts, or phrasing to copy.

# Layering and splitting files

| Layer | Content | Requirement |
|---|---|---|
| Metadata | name, description | Short; distinguishable from neighboring skills |
| SKILL.md | Core judgment, necessary workflow, key constraints, completion criteria | Enough to start the task and know what to look up next |
| Resources | references, scripts, assets | Read or executed only under specific conditions |

- The criterion for splitting is "is this content only needed in specific situations", not line count. A body over roughly 500 lines usually means conditional content has crept in; treat that as a warning line.
- Write references as "when X, read Y", not "read everything before starting". Prefer shallow references; avoid cycles and long mandatory chains. A reasonable second-level reference is fine. Add a table of contents to large files.
- For multi-domain skills, split references by domain so the model reads only the matching one.
- A simple skill can be a single SKILL.md.

# Scripts

Worth extracting: the same logic keeps being rewritten, the rules are stable, manual execution is error-prone, and repeatability clearly improves reliability. First check whether existing tools already do it reliably; add a script only when there is real payoff.

Write the calling contract in the body:

```markdown
Run `scripts/xxx.py`:
- Input: ...
- Output: ...
- Side effects: ...
- Failure signal: non-zero exit code; error message explains ...
```

- Execute scripts by default rather than reading their source; read source only to debug or modify.
- Never swallow exceptions and return success.
- New or modified scripts should be verified against the behavior they affect, preferring isolated data, a test environment, or dry-run. Real side effects (sending email, deleting resources, deploying, paid API calls) remain bound by the user's authorization; "to verify it" is not extra authorization. For parts that can't be executed safely, state which checks were done and what remains unverified.
- Where writes may be repeated, consider whether retries produce duplicates (idempotency).

# Minimal skeleton

Trim as needed. No skill needs every section, and filling a section just to complete the template dilutes the signal:

```markdown
---
name: skill-name
description: <specific capability>. <task conditions where it applies>. <only if easily confused: not for ...>
---

# Goal and completion criteria
What is delivered, and what observable evidence confirms completion.

## Required inputs
How to obtain missing ones, or when clearly labeled assumptions are acceptable.

## Procedure
Only steps with real ordering or checkpoints; for open-ended tasks, write judgment criteria instead.

## Key constraints
- <constraint> — <reason, when needed>

## Failure handling
- <realistic key failures, recovery, or stop conditions>

## Resources on demand
- When <condition>, read `references/xxx.md`.
- When <operation> is needed, run `scripts/xxx.py` (input / output / side effects / failure signal).

## Deliverable
Artifacts returned, verification results, open issues; a short template if a format is required.
```

Directory:

```text
my-skill/
├── SKILL.md
├── references/   # notes needed only in specific situations
├── scripts/      # execution logic worth reusing
└── assets/       # templates and materials used in outputs
```

# Final self-check

1. Can the model accurately judge when to use it?
2. Does every passage pass the keep-or-cut test?
3. Can the needed resources be located without reading everything?
4. Are scope, key failure handling, and completion evidence explicit?
5. Is there an observable quality or efficiency gain on real tasks? (Without comparison data, say honestly that this is unverified.)

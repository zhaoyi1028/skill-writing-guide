# Contributing

## Issues

The most useful issue is a **failure case**. Please include:

1. Your request, verbatim (redacted as needed).
2. Platform and model.
3. What the skill actually did (key steps, files read, output produced).
4. What you expected it to do.
5. Which layer you think failed: not triggered / wrong path / operation failed / wrong result declared complete / did more than asked.

"This passage reads badly" is welcome too, but say in what situation the model would do better with it removed or rewritten.

## Pull requests

This skill follows its own principles. Check your PR against them:

- **Keep-or-cut test**: for every added passage, would the model be more likely to err without it? If not, don't add it.
- **No prohibitions appended at the end**: fixes go where the problem lives.
- **Reasons, not shouting**: constraints that are easy to misread get one sentence of why.
- **No keyword stuffing**: when editing the description, consider both recall and false triggers.
- **Layering**: content needed only in specific situations goes in `references/`; the body stays just enough to start the task.
- **No unverified empirical claims** (e.g. "model compliance drops for X").

In the PR description, state what changed, why, and how it was verified (static check / behavioral test / unverified).

## Testing

There is no automated test suite yet. For manual verification, cover at least these requests and record actual behavior:

| Request | Expected behavior |
|---|---|
| "Audit this skill, opinions only" | Findings with location and evidence; no files modified |
| "Only tighten the description, leave the rest" | Only the description changes |
| "The skill isn't triggering, help me locate why" | Checks discovery, selection, execution in turn; doesn't blame the description by default |
| "Add verification for a real email-sending script" | Verifies in isolation; doesn't send test emails on its own |
| "Merge these files with the existing PDF skill" | Does not trigger this skill |

If you've run a comparison with Claude Code's `claude plugin eval` or the skill-creator eval flow, please submit cases and results under `evals/`.

## Validation

Before submitting, run the structural check (script from Anthropic's skill-creator):

```bash
python -m scripts.quick_validate path/to/skill-writing-guide
```

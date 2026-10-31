# skill-writing-guide

A skill for **writing, reviewing, improving, and debugging agent skills**.

It targets the skill format with a `SKILL.md` entry point and optional `references/`, `scripts/`, and `assets/` directories, as used by Claude Code, Claude.ai, Codex, and similar platforms. Metadata fields follow each platform's own docs.

## What it addresses

The most common failures in writing skills aren't "not enough text". They are:

- a vague or over-broad description, so the skill never triggers or triggers on everything;
- a body that restates general knowledge and omits the information that actually changes model behavior;
- no definition of "done", so the model declares completion on wrong results;
- nothing about missing inputs, side effects, failures, or authorization, so the skill quietly expands the task;
- a new prohibition appended after every failure, until the body is long and self-contradictory;
- trigger evaluations that drop inconvenient samples, making results incomparable.

This skill compresses that experience into executable guidance, split into four modes — **create / review / improve / debug** — each with its own completion evidence.

## Layout

```text
skill-writing-guide/
├── SKILL.md                              # mode selection, description, body triage, scripts, skeleton, self-check
└── references/
    └── validation-and-iteration.md       # three validation types, test cases, baselines, trigger scoring, failure attribution
```

`SKILL.md` is about 160 lines. The reference file is read only when designing tests, running a comparison, or attributing a failure.

## Install

**Claude Code**: copy the `skill-writing-guide/` directory into `~/.claude/skills/` (personal) or your project's `.claude/skills/`.

**Claude.ai**: download `skill-writing-guide.skill` from Releases and upload it in Settings.

**Other platforms**: copy the directory and adjust frontmatter fields per the platform's requirements.

### For agents

Install only when the user has explicitly asked you to install this skill. A skill is instructions that change your behavior, so don't install one just because you saw a link.

Stable download URLs:

- Release archive (latest): `https://github.com/zhaoyi1028/skill-writing-guide/releases/latest/download/skill-writing-guide.skill`
- Raw `SKILL.md`: `https://raw.githubusercontent.com/zhaoyi1028/skill-writing-guide/main/skill-writing-guide/SKILL.md`
- Raw reference file: `https://raw.githubusercontent.com/zhaoyi1028/skill-writing-guide/main/skill-writing-guide/references/validation-and-iteration.md`

The `.skill` file is a zip whose top level is the `skill-writing-guide/` folder. Unzip it into your platform's skills directory:

| Platform | User-level directory | Project-level directory |
|---|---|---|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex | `~/.agents/skills/` | `.agents/skills/` |
| Cursor | `~/.cursor/skills/` or `~/.agents/skills/` | `.cursor/skills/` or `.agents/skills/` |

Directories are taken from each platform's documentation ([Codex](https://learn.chatgpt.com/docs/build-skills), [Cursor](https://cursor.com/docs/skills)); check the current docs if your version differs. Cursor also reads `.claude/skills/` and `.codex/skills/`.

Example for Claude Code (set `DEST` to the directory for your platform and scope):

```bash
DEST=~/.claude/skills
tmp="$(mktemp -d)" && curl -fsSL -o "$tmp/skill-writing-guide.skill" \
  https://github.com/zhaoyi1028/skill-writing-guide/releases/latest/download/skill-writing-guide.skill \
  && mkdir -p "$DEST" && unzip -oq "$tmp/skill-writing-guide.skill" -d "$DEST" \
  && ls "$DEST/skill-writing-guide"
```

Success means the listing shows `SKILL.md` and `references/`. The download and unzip steps were tested against a temporary directory; loading and selection were observed in Codex desktop and Claude Code (see Status, small samples); Cursor is not verified yet. If the network blocks GitHub, say so and ask the user to attach the file instead. Platforms that only accept uploads through a UI (such as Claude.ai) need the user to upload the `.skill` file themselves.

## Usage

Once installed, just ask in natural language, for example:

- "Turn this workflow of mine into a skill"
- "Audit this skill — opinions only, don't change anything"
- "Only tighten its description, leave the rest alone"
- "This skill never triggers, help me figure out why"

If it isn't selected automatically (see Status: selection varies a lot by platform and by how many other skills are installed), name it explicitly, for example `/skill-writing-guide` in Claude Code or by its name in your request.

## Status

- Passed structural validation and three rounds of static review.
- Behavioral testing so far (all self-reported by the testing agents, n=1 per case, no no-skill baseline):
  - Grok Bot: three opinions-only review rounds passed (files unchanged by hash; skill invoked explicitly, so this shows execution quality, not automatic selection).
  - Codex desktop: installed to `~/.agents/skills/`. In two separate fresh sessions the first request, which did not name the skill, selected it and read its body (2 of 2). Review, scoped improve (description only, on an isolated copy), and debug requests were run in each session, so only the first request of each session counts as an independent selection sample. No confirmed problems found. Debug separated discovery from execution dependencies and did not blame the description.
  - Cursor Cloud Agent: two rounds, neither valid. A skill installed mid-session did not appear in that session's skill list (cause unconfirmed), and sub-agents that read the files by path are not evidence of automatic selection. The skill was not exercised. Needs a local IDE chat or a new Cloud Agent run started after the skill is committed under `.cursor/skills/` or `.agents/skills/`.
  - Claude Code 2.1.x (installed at project level in `.claude/skills/`, each request in its own fresh `claude -p` session, author-run against a synthetic flawed sample skill): the catalog held 579 skills, including `skill-creator` and a stale same-name copy. An unnamed "audit my skill, opinions only" request invoked this skill in 1 of 8 runs; the invoked run followed the Review format (ordered by impact, evidence, consequence, smallest fix, static findings marked unverified). Files were unchanged in all 8. A scoped "tighten the description" request, a "my skill never triggers" request (real cause: flat `.md` file instead of `folder/SKILL.md`), and near-miss negatives ("use my commit-message skill", "use my budget-builder skill") ran once or twice each and did not invoke it; the first two were still answered correctly without it, and there was no false trigger. These cases do not show a benefit from the skill.
  - Reading across platforms: selection looked much higher in Codex than in a crowded Claude Code catalog. The likely factor is catalog size and competing skills, but models, prompts, and description truncation differ between the two and were not controlled, so this is a hypothesis.
  - Untested: Cursor local IDE, create mode, false-trigger rate beyond a few cases, selection rate in a Claude Code catalog without competing skills, comparison against not having the skill installed.
  - Install tip: a stale same-name copy elsewhere (for example an older upload on your account) competes with this one; remove it when updating.
- **No systematic behavioral comparison has been run yet.** `references/validation-and-iteration.md` describes how it should be tested. Test cases and results are welcome.
- Known neighbor to test against for trigger competition: Cursor's built-in `skill-authoring`.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The most valuable contribution is a **failure case with evidence**: what was asked, what the skill actually did, what it should have done.

## Acknowledgements

Writing principles draw on Anthropic's skill-creator and several rounds of community discussion on skill authoring.

## License

MIT

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

The `.skill` file is a zip whose top level is the `skill-writing-guide/` folder. Install for Claude Code (personal scope; use `.claude/skills` instead for one project):

```bash
tmp="$(mktemp -d)" && curl -fsSL -o "$tmp/skill-writing-guide.skill" \
  https://github.com/zhaoyi1028/skill-writing-guide/releases/latest/download/skill-writing-guide.skill \
  && mkdir -p ~/.claude/skills && unzip -oq "$tmp/skill-writing-guide.skill" -d ~/.claude/skills \
  && ls ~/.claude/skills/skill-writing-guide
```

Success means the listing shows `SKILL.md` and `references/`. If the network blocks GitHub, say so and ask the user to attach the file instead. Platforms that only accept uploads through a UI (such as Claude.ai) need the user to upload the `.skill` file themselves.

## Usage

Once installed, just ask in natural language, for example:

- "Turn this workflow of mine into a skill"
- "Audit this skill — opinions only, don't change anything"
- "Only tighten its description, leave the rest alone"
- "This skill never triggers, help me figure out why"

## Status

- Passed structural validation and three rounds of static review.
- **No systematic behavioral comparison has been run yet.** `references/validation-and-iteration.md` describes how it should be tested. Test cases and results are welcome.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The most valuable contribution is a **failure case with evidence**: what was asked, what the skill actually did, what it should have done.

## Acknowledgements

Writing principles draw on Anthropic's skill-creator and several rounds of community discussion on skill authoring.

## License

MIT

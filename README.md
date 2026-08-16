# Pair Me

> Portable AI skills for any tool. Drop one in. Your AI levels up.

A growing collection of plug-and-play skills for Claude Code, Cursor, GitHub Copilot, Windsurf, Cline, and OpenAI Codex. Each skill is a single markdown file — no dependencies, no frameworks, no lock-in. Just copy it in and go.

---

## Why this exists

AI tools are powerful. But the prompts behind them are usually scattered, duplicated, and written once then forgotten. These skills are built to be **reusable, portable, and consistent** — the same behavior whether you're on Claude Code today or Cursor tomorrow.

Every skill follows the same format:
- Pure imperative prose — no tool-specific syntax
- Portable frontmatter (`skill`, `description`, `input`, `output`, `phase`)
- Works standalone — paste into any AI tool and it runs

---

## Categories

Skills are grouped by who uses them day-to-day.

### [engineers](skills/engineers/)

Software engineering workflows — orient in a codebase, plan a feature, choose the structure it needs, implement it phase by phase, define test cases, document what shipped, and commit cleanly. Sixteen skills covering the full loop from `onboard-project` to `create-pr`.

See [skills/engineers/README.md](skills/engineers/README.md) for the full list and install commands.

### [utilities](skills/utilities/)

Cross-cutting skills for managing the AI tool or session itself — not tied to any one profession.

See [skills/utilities/README.md](skills/utilities/README.md) for the full list and install commands.

More categories (design, product, ops) will land here as the collection grows.

---

## Install skills

Browse everything available in this repo:
```bash
npx skills add pdkproitf/skills --list
```

Install a specific skill by name:
```bash
npx skills add pdkproitf/skills@<skill-name>
```

Install several at once:
```bash
npx skills add pdkproitf/skills --skill <skill-name> --skill <skill-name>
```

Install every skill in the repo:
```bash
npx skills add pdkproitf/skills --all
```

For global installation (all projects):
```bash
npx skills add pdkproitf/skills --global
```

---

## Skill format

Every skill in this repo follows the same portable format:

```markdown
---
skill: skill-name
description: one line — what this skill does
input: what the caller passes in
output: what this skill produces
phase: orient | research | plan | implement | validate | commit
---

# skill-name

## When to trigger
## Step 1 — ...
## Step 2 — ...
```

No `$ARGUMENTS`. No tool names. No slash command cross-references. Just instructions any AI can follow.

---

## Contributing

Skills should be:
- **Self-contained** — no external dependencies or imports
- **Tool-agnostic** — no Claude-specific syntax, tool names, or slash commands
- **Focused** — one clear job, one clear output
- **Idempotent** — safe to run more than once

Open a PR with your skill in `skills/{category}/{your-skill-name}/` alongside a `README.md` entry for that category. The skill file (`.md`) and metadata are all that's needed — npx skills handles installation for all tools.

---

## More

- [CHANGELOG.md](CHANGELOG.md) — release history
- [MIGRATION.md](MIGRATION.md) — upgrading from `install.sh` to `npx skills`

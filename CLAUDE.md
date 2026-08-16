# Pair Me — Portable AI Skills Repository

## What this is

A growing collection of portable, tool-agnostic AI skills that work across Claude Code, Cursor, GitHub Copilot, Windsurf, Cline, and other AI tools. Each skill is a single markdown file following the [agentskills.io specification](https://agentskills.io/specification) — no dependencies, no frameworks, no lock-in.

Skills are organized by audience:
- **`skills/engineers/`** — software engineering workflows (16 skills covering planning, design, implementation, documentation, testing, committing)
- **`skills/utilities/`** — cross-cutting tools for managing the AI session itself
- **`agents/`** — full multi-step agents that can run autonomously or in response to user requests

## Project structure

```
.
├── README.md                          # Main overview
├── CHANGELOG.md                        # Release history
├── MIGRATION.md                        # Upgrade guide (install.sh → npx skills)
│
├── skills/
│   ├── engineers/                     # For software engineers
│   │   ├── central-workspace/         # Bootstrap workspace config
│   │   ├── onboard-project/           # Load project context at session start
│   │   ├── codebase-indexing/         # Build/refresh code graph indexes
│   │   ├── locate-code/               # Find WHERE code lives
│   │   ├── analyze-code/              # Understand HOW code works
│   │   ├── find-patterns/             # Extract examples and conventions
│   │   ├── design-patterns/           # Choose, apply, or audit design patterns
│   │   ├── feature/                   # Research & write feature spec
│   │   ├── implement/                 # Execute spec phase by phase
│   │   ├── define-test-case/          # Define acceptance tests (DSL), per implement phase
│   │   ├── commit/                    # Group changes & write commit messages
│   │   ├── document/                  # Generate feature docs from diff
│   │   ├── save-progress/             # Checkpoint: WIP commit + session file
│   │   ├── resume-work/               # Restore session and continue
│   │   └── architecture/              # Map system into docs_context layer
│   │
│   └── utilities/
│       └── token-wake/                # Preserve Claude Pro token window
│
└── agents/
    ├── codebase-indexer.md            # Run `codebase-indexing` skill
    └── committer.md                   # Run `commit` skill autonomously
```

## Skill format

Every skill is a single markdown file with:

```markdown
---
name: skill-name
description: one line — what this skill does
metadata:
  phase: "orient | research | plan | implement | validate | commit"
  input: "what caller passes in"
  output: "what skill produces"
  dependencies: "other skills or tools this invokes"
---

# skill-name

## When to trigger
[Conditions and use cases]

## Step 1 — [Phase name]
[Imperative instructions]

## Step 2 — ...
```

**Conventions:**
- Lowercase hyphenated `name` must match parent directory
- Pure imperative prose — no tool-specific syntax (no `/command`, no `$ARGUMENTS`)
- Tool-agnostic — instructions work in any AI tool
- Structured steps with clear inputs/outputs
- Each skill is self-contained and idempotent

## The engineering workflow

Skills form a pipeline. Typical feature flow:

1. **`onboard-project`** — Load docs, domain models, TODOs, refresh code indexes (session start)
2. **`feature`** — Research codebase, design options, write spec (incl. the behaviors that must hold under test)
3. **`implement`** — Execute spec phase by phase, verify, commit each phase
4. **`define-test-case`** — Invoked by `implement` at the start of each phase: turns that phase's behaviors into seam-anchored DSL cases before its code is written
   - **`design-patterns`** — Invoked by `feature` on a structural design option, and again by `implement` per phase when the shape of a service is open; also runs standalone to audit existing code
5. **`commit`** — Group changes by logical concern, write Conventional Commit messages
6. **`save-progress`** — Checkpoint: WIP commit + session file (if interrupted)
7. **`resume-work`** — Restore session and continue from last unchecked step
8. **`document`** — Generate docs from diff, register in `doc_dictionary.md`
9. **`architecture`** — Periodic full-system scan to keep docs fresh

On-demand research skills:
- **`locate-code`** — Find WHERE code lives (file paths by layer)
- **`analyze-code`** — Understand HOW code works (data flow, logic, dependencies)
- **`find-patterns`** — Extract working examples and conventions
- **`design-patterns`** — Choose, apply, or audit design patterns (GoF + modern + architectural)
- **`codebase-indexing`** — Build/refresh structural and semantic code graphs (one writer, many readers)

See [skills/engineers/README.md](skills/engineers/README.md) for the full flowchart.

## Key decisions & philosophy

### Tool agnostic, not tool-specific
- Skills describe work, not tool commands. They run the same way in Claude Code, Cursor, or Windsurf.
- No slash-command references, no tool assumptions, no hardcoded paths.
- Paths and tool details go in the workspace config (e.g. `CLAUDE.md`, `.cursorrules`), which the skill reads.

### One graph, many readers
- Only `codebase-indexing` builds indexes. It's the single writer.
- `locate-code`, `analyze-code`, `find-patterns`, and `architecture` only *read* the graph.
- Readers fall back to manual grep/search immediately if the index isn't ready — they never block on a cold build.
- Keeps the architecture simple and predictable: no surprising re-indexes mid-task.

### Portable format
- Skills are `.md` files, not YAML, JSON, or proprietary formats.
- agentskills.io spec compliance means they stay valid if installed in other collections.
- No `require()`, no imports, no framework assumptions.

### Idempotent steps
- Each skill is safe to run more than once.
- `onboard-project` can run 10 times without side effects.
- `codebase-indexing` detects freshness and skips unnecessary rebuilds.

## Working on this repo

### Adding a new skill

1. Create a directory: `skills/{category}/{skill-name}/`
2. Add `SKILL.md` (following the format above) and `README.md` (a one-paragraph overview for the README index)
3. Update the category `README.md` to list the new skill
4. Open a PR — npx skills will auto-discover and install it

Example PR: add `skills/engineers/my-skill/SKILL.md` + update `skills/engineers/README.md`

### Editing an existing skill

- Edit the `.md` file directly
- Commit with a clear message: `refactor(skill-name): description`
- Skills are versioned by commit — no separate version file needed
- Updates land automatically when users run `npx skills add`

### Testing a skill locally

1. Copy the skill file into your project's `.claude/skills/` or equivalent
2. Invoke it via `/skill-name` in Claude Code, or paste it directly
3. Iterate on the steps, test in a real project, refine instructions
4. Once working, open a PR here

### Versioning & releases

- We follow [Conventional Commits](https://www.conventionalcommits.org/)
- Releases are tagged as `v1.2.3` on main
- `CHANGELOG.md` documents breaking changes, new skills, and major improvements
- npx skills handles version selection; projects auto-update when requested

## Installing skills

Users install from this repo via [npx skills](https://www.npmjs.com/package/@agentskills/cli):

```bash
# Install all skills
npx skills add pdkproitf/skills --all

# Install a specific skill
npx skills add pdkproitf/skills@onboard-project

# Global installation (all projects)
npx skills add pdkproitf/skills --global

# List everything available
npx skills add pdkproitf/skills --list
```

## Contributing guidelines

Skills should be:
- **Self-contained** — no external dependencies or imports
- **Tool-agnostic** — no Claude-specific syntax, tool names, or slash commands
- **Focused** — one clear job, one clear output
- **Portable** — work in any AI tool (Claude Code, Cursor, Copilot, Windsurf, Cline, etc.)
- **Idempotent** — safe to run multiple times without side effects
- **Tested** — walk through the steps in a real project before submitting a PR

Open a PR with:
- `skills/{category}/{your-skill-name}/SKILL.md` (the skill itself)
- `skills/{category}/{your-skill-name}/README.md` (one-paragraph overview)
- Updated `skills/{category}/README.md` (add an entry for the new skill)

## Maintenance & operations

### Building the code graph
```bash
# Status check
npx codebase-memory search-graph --project <name>

# Force rebuild (rare)
npx codebase-memory index-repository --repo-path . --force
```

### Reviewing changes
```bash
# See what's changed since last commit
git log --oneline -n 20

# Diff a specific skill
git diff HEAD^ skills/engineers/my-skill/SKILL.md
```

### Release workflow
1. Update `CHANGELOG.md` with new/changed skills and breaking changes
2. Commit: `chore(release): v1.2.3` (bumping version in CHANGELOG)
3. Tag: `git tag v1.2.3 && git push origin main --tags`
4. npx skills picks up the tag automatically

## Common tasks

| Task | How |
|------|-----|
| **Add a new skill** | Create `skills/category/skill-name/SKILL.md` + `README.md`, update category index |
| **Fix a skill** | Edit `.md`, test locally, commit with `fix(skill-name): ...`, open PR |
| **Rename a skill** | Update all references in `README.md`s, rename directory, commit with `refactor(skills): rename ...` |
| **Remove a skill** | Delete directory, update `README.md`, commit with `chore(skills): remove ...` |
| **Update docs** | Edit `.md` files in `skills/*/` or top-level `README.md` / `CHANGELOG.md` |
| **Install in a project** | User runs `npx skills add pdkproitf/skills@skill-name` in their project |
| **Check what's installed** | User runs `npx skills list` in their project |

## Links

- **[Main README](README.md)** — project overview, install instructions
- **[Engineering Skills](skills/engineers/README.md)** — full list, workflow diagram, detailed descriptions
- **[Utility Skills](skills/utilities/README.md)** — session management tools
- **[Changelog](CHANGELOG.md)** — release history, breaking changes
- **[Migration Guide](MIGRATION.md)** — upgrade from `install.sh` to `npx skills`
- **[agentskills.io](https://agentskills.io/specification)** — skill format spec

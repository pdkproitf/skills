---
name: onboard-project
description: Prime — load project context by reading docs, the docs_context layer, core docs, and TODO before starting any task
metadata:
  phase: "orient"
  input: "no arguments — invoke as-is"
  output: "summary of the codebase covering domain models, key workflows, and active work areas"
  dependencies: "central-workspace (invoked if no workspace file is bootstrapped yet), codebase-indexing (index refresh), architecture (invoked if docs_context/system_context are missing)"
---

# Prime

Execute every section below **in order — Workspace → Index → Read → Research (if triggered) → Self-check → Report**
— to build a complete understanding of the codebase. Do not jump to the user's task before Self-check
confirms every step ran; a step that doesn't apply must still be explicitly marked N/A with a reason,
not silently dropped.

## When to trigger

Use this skill when:
- this is a new session and project context has not yet been loaded
- the user asks to "prime" or "load context" before starting work
- another skill lists `onboard-project` as a dependency and context isn't loaded yet (invoke automatically)

## Step 1 — Workspace (bootstrap if needed)

Check whether a `# WORKSPACE` section is already loaded in context (the auto-load file every
tool loads at session start — `CLAUDE.md`, `.cursorrules`, etc. — would have pulled it in before
this skill ever runs). Do not re-derive tool detection or path conventions here — that logic
belongs solely to `central-workspace`.

- **`# WORKSPACE` present** — a workspace file already exists for this tool. Proceed to Index.
- **`# WORKSPACE` absent and the `central-workspace` skill is available** — this project hasn't
  been bootstrapped yet. Invoke `central-workspace` in full mode before continuing, so the config
  keys the rest of this skill relies on (`docs_context`, `system_context`, `docs_dictionary_file`,
  `todo_file`, etc.) resolve to real, project-specific paths instead of skill-level defaults.
- **`# WORKSPACE` absent and `central-workspace` is unavailable** — proceed with every config key
  at its documented default and note in the Report that no workspace is configured, recommending
  `central-workspace` be installed.

## Step 2 — Index (if available)

Invoke the **`codebase-indexing` skill** to ensure the code graph indexes are built and fresh
before the reading skills below run. That skill is the single writer — it checks status cheaply
and builds only if missing or stale, so this is safe and near-instant on a warm repo. Do not
call `index_repository` or `graphify reflect` directly here.

If neither index backend is available, `codebase-indexing` reports so and the downstream skills
fall back to manual search — proceed to Read either way. For a very large, unindexed repo, run
`codebase-indexing` as a background agent so the build doesn't block orientation.

Fresh indexes accelerate `locate-code`, `find-patterns`, `analyze-code`, and enrich the
`architecture` skill's output.

If `codebase-indexing` reports `codebase-memory-mcp` is available, note in the Report that the user
may want standing permission for its read-only tools (`search_graph`, `get_code_snippet`,
`get_architecture`, `query_graph`, `search_code`, `trace_path`, `index_status`, `list_projects`,
`get_graph_schema`) — e.g. in Claude Code, an allow rule for `mcp__codebase-memory-mcp__*` read/list/get
calls via the `update-config` skill, if available, so later orientation runs don't prompt per call.
This is a suggestion only — never edit permission/settings files yourself here.

## Step 3 — Read

This skill **executes** the reading protocol; it doesn't define it. The protocol —
tiers, reader roles, matching and budget rules — lives in `# WORKSPACE` → **Context Loading**,
which is auto-loaded every session and is authoritative. Follow it rather than re-deriving it here.

1. Read `docs_dictionary_file` (default: `.docs/doc_dictionary.md`) — the map. Its `## Core Context`
   section lists what to load for this reader role; `## Features` is the keyword-matched layer.
2. Load **Tier 0** for the *in-repo agent* role, exactly as Context Loading defines it. Also read
   `README.md` for what/who — Tier 0 is authoritative where they disagree.
   - An older `.context/` directory layer, or a monolithic file with no `system_context` sibling,
     may still exist — read what's there and recommend re-running `architecture` to migrate.
3. Load **Tier 1** for the current task by matching the dictionary's `## Features` entries, honoring
   Context Loading's match rules and cap.
4. If `todo_file` doesn't exist, create it with a minimal template (a title and an empty task list) —
   a trivial write, not a scan, so it isn't gated behind Research below.

If the Tier 0 docs are absent but `README.md` or `docs_dictionary_file` already exist, don't generate
them here — invoke the **`architecture` skill** directly to backfill `docs_context`/`system_context`.
Enough context already exists from `README.md`/the dictionary to seed it, so the full Research pass
below isn't needed for this case. Note in the Report that `architecture` ran to fill the gap. Only
missing docs across the board triggers Research below.

## Step 4 — Research (conditional)

If both `README.md` and `docs_dictionary_file` do not exist, trigger Research; otherwise skip it
— but record the skip in Self-check below, don't just drop it.

Use these tools in sequence to build a full picture before doing any work. All of the following skills will use `codebase-memory-mcp` if indexed (see Index step above):

1. **`locate-code` skill** — map where key concepts live (models, services, jobs, controllers)
   - Run for the main domain concepts found in the README
   - Output: file paths grouped by layer — no content reading yet

2. **`analyze-code` skill** — understand architecture and how components fit together
   - Run for the overall system and any domain areas flagged in the docs
   - Output: big-picture understanding, data flow, component relationships

3. **`architecture` skill** — map the system into `docs_context` so future sessions don't repeat this research from scratch
   - Output: `docs_context` (business content) and its sibling `system.md` created

## Step 5 — Self-check (definition of done)

Before writing the Report, confirm each item below — either checked off or explicitly marked N/A
with the reason. Do not proceed to Report, and do not start the user's actual task, until every
item is accounted for:

- [ ] Workspace: `# WORKSPACE` confirmed present, or bootstrapped via `central-workspace`, or its
      absence was noted (`central-workspace` unavailable)
- [ ] Index: `codebase-indexing` invoked (or its unavailability noted)
- [ ] Read: `docs_dictionary_file` read, Tier 0 loaded, Tier 1 matched, `todo_file` exists/created
- [ ] Gap-fill: if Tier 0 was absent but `README.md`/`docs_dictionary_file` existed, `architecture`
      was invoked directly (Step 3's gap-fill case) — or this case didn't apply
- [ ] Research: triggered and completed if both `README.md` and `docs_dictionary_file` were absent
      — or explicitly skipped because at least one existed
- [ ] Permission suggestions from Step 2 (if any) captured for the Report

If any box can't be checked because the step was actually skipped by mistake, go back and run it
now — don't write it up as done.

## Step 6 — Report

Keep this a compact orientation summary — reference what each doc said, don't restate it in full. Cover:
- What the app does and who uses it
- System structure — key layers and modules (from `system.md`, if loaded)
- Key domain models and their relationships
- Main workflows (e.g. how a video gets published)
- Active work areas (from `todo_file`)
- Suggested permission updates, if any (e.g. standing read/list/get access for `codebase-memory-mcp` tools)

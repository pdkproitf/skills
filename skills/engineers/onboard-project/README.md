# onboard-project

Load project context before starting any task — prime the AI with domain models, docs, active work, and conventions for a complete mental model.

---

## What it does

**Reads documentation in order:**
- **[Optional] Index the codebase** — builds code graph if `codebase-memory-mcp` available and not yet indexed (accelerates downstream search)
- **`README.md`** — what the app is and who uses it
- **`docs/CONTEXT.md`** — domain models, key workflows, layers, patterns, external dependencies (maintained by `architecture` and `document`)
- **`docs/core/*.md`** — conditional load: only entries whose keywords match current task (via `doc_dictionary.md`)
- **`docs/TODO.md`** — active work areas (created empty if missing)

**Fallback: code exploration (if no docs exist)**
- Uses `locate-code` to map where key concepts live
- Uses `analyze-code` to understand how they fit together
- Uses `architecture` to write `docs/CONTEXT.md` for future sessions
- If only `CONTEXT.md`/`system.md` are missing (README/dictionary already exist), invokes `architecture` directly to backfill them — skips the heavier `locate-code`/`analyze-code` pass since enough context already exists to seed it

**Report generated:**
- Compact orientation summary (not a restatement of every doc)
- Domain models and relationships
- System structure (layers, modules)
- Key workflows
- Active work areas

---

## When to use

- At session start before implementing or planning
- After switching to unfamiliar codebase area
- As a dependency — other skills (`feature`, `implement`) invoke automatically

---

## Install

```bash
npx skills add pdkproitf/skills@onboard-project
```

---

## Usage

**Claude Code:**
```
/onboard-project
```

**Other tools:**
```
@onboard-project
```

**No arguments needed** — reads project context and reports automatically.

---

## Output

**Structured summary covering:**
- What the app does and who uses it
- System structure (layers, modules)
- Key domain models and relationships
- Main workflows (how features flow end-to-end)
- Active work areas from `docs/TODO.md`

# architecture

> Map the overall system architecture from a full codebase scan into a `docs_context` documentation layer.

---

## What it does

`architecture` scans **the current repo** as a whole — not a diff — and writes a purpose-split
documentation layer, split by who reads it rather than into one growing monolith:

- **`docs_context`** (default `.docs/CONTEXT.md`) — business flows, journeys, concepts, owned capabilities; PM-readable. **Authored** (reconciled on re-run). Optionally ends with a **System at a Glance** mermaid diagram — components and their runtime as nodes, transport and payload on the arrows (`HTTPS · /orders`, `AMQP · order.created`). It's the one opt-in section in the layer: the skill asks once on a fresh run, defaults to no, and never adds it unprompted on a reconcile.
- **`system.md`** (sibling) — layers, data flow, domain models, invariants, file index, API surface, outbound dependencies, event contracts; for developers and AI agents. **Authored** (reconciled on re-run) — this is the single source of technical truth.
- **`service-manifest.md`** — a portable, agent-ready registry entry: who this service is, what contracts it **publishes**, and what it **consumes** from other services (by name, with spec pointers — never inlined schemas). **Projected**, not authored — every field traces back to something already in `system.md`, plus git identity. It exists purely so an orchestrator can read one small file per repo instead of every repo's full `system.md`.

**One repo per run, multi-repo-ready output.** The skill only ever scans the repo it runs in, but
the manifest is designed so a separate **multi-repo orchestrator** can glob every repo's
`.docs/service-manifest.md`, join their frontmatter into a platform service registry, and wire the
cross-repo dependency graph — without re-scanning any code. The orchestrator node itself
(`context-orchestrator.md`) is assembled by that orchestrator run, *not* by a per-repo scan.

**This is the knowledge layer, not a plan.** It documents what exists and what contracts a
service honors. Deciding what to build next — in this repo or one that depends on it — is
`feature` (plan) and `implement` (build); they read these docs, plus any dependency's
`service-manifest.md`, as starting context.

Unlike `document`, which documents a single implemented feature, this skill maps the system broadly.
It reads graph indexes via the `codebase-indexing` skill (the single writer) rather than building
them itself.

Re-running `CONTEXT.md`/`system.md` is a reconciliation, not a rewrite: it reads each existing
file first, keeps what still matches reality, updates what's drifted, and adds what's new —
including preserving any hand-written content that doesn't fit the template. `service-manifest.md`
is handled differently — it's re-projected wholesale from the freshly reconciled `system.md`
rather than diffed, with one exception: hand-added `consumes` entries (dependencies no scan can
see, e.g. via a shared library) are always carried forward. Each file has its own template (under
the skill's `templates/` directory) with a line-cap and required sections, checked by a
self-verify pass before the skill reports done.

It also compares its findings against `README.md` and flags mismatches (stale claims,
undocumented capabilities) in its report — but never edits `README.md` directly. That file is
human-voiced prose; only its author decides wording and what's worth mentioning.

---

## When to use

- Bootstrapping the `docs_context` layer for a project that doesn't have any yet
- Periodically, to correct drift that `document`'s incremental per-feature patches don't catch
- Before a major refactor, to get an accurate picture of current structure

---

## Install

```bash
npx skills add pdkproitf/skills@architecture
```

---

## Usage

**Claude Code:**
```
/architecture
```

**Other tools:**
```
@architecture
```

No arguments needed. The skill scans the current codebase and reports automatically.

---

## Output

```
{docs_context}                      (default: .docs/CONTEXT.md — business content, authored)
{dirname(docs_context)}/system.md   (technical content, authored — interface sections omitted if none found)
{dirname(docs_context)}/service-manifest.md   (portable multi-repo aggregation unit, projected from system.md)
```

`{docs_context}` defaults to `.docs/CONTEXT.md` and is a single file, not a directory —
`system.md` and `service-manifest.md` are written alongside it. If an older layer exists (a
`.context/` directory, or a monolithic single file with no `system.md`), re-running the skill
migrates it into this shape and removes the old structure.

Reports whether this was a fresh generation or a reconciliation, and for each file: created,
reconciled (with what changed), unchanged, or skipped (with why). If `README.md` looks stale
compared to the scan, lists those mismatches too — as suggestions, not changes already made.

---

## Dependencies

- Invokes `codebase-indexing` (the single index writer) at the start of its scan to ensure the codebase-memory-mcp and graphify indexes are fresh; reads them read-only thereafter
- Uses `locate-code` and `analyze-code` internally to map structure
- Each output file's exact markdown skeleton lives in this skill's `templates/` directory (`context.md`, `system.md`, `service-manifest.md`, and `context-orchestrator.md` for the aggregating orchestrator)
- Reads `docs_context` from the project's `workspace.md` if `central-workspace` has been run; otherwise defaults to `.docs/CONTEXT.md`

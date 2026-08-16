---
name: architecture
description: Map the whole system into a `docs_context` documentation layer from a full codebase scan — creates or reconciles the business and system docs. Supports single-service repos and multi-service monorepos with orchestrator node.
metadata:
  phase: "orient"
  input: "no arguments — scans the current codebase"
  output: "three artifacts for the current repo — CONTEXT.md (business), system.md (technical), and service-manifest.md (portable, multi-repo-aggregation unit)"
  dependencies: "codebase-indexing, design-patterns, codebase-memory-mcp, graphify"
---

# Map Architecture

Scan **the current repo** as a whole — not a git diff — and produce a `docs_context` layer that
agents load at session init. Unlike `document`, which documents one feature from its diff, this
skill maps the system broadly and is meant to be re-run periodically so the layer never drifts
from reality.

**One repo per run, multi-repo-ready output.** A run scans only the current repo, but produces a
portable **service manifest** alongside the docs so a multi-repo orchestrator can aggregate this
repo's knowledge with others' — without ever re-scanning code. Three artifacts:

| File | Answers | Read by |
|---|---|---|
| `docs_context` (default: `.docs/CONTEXT.md`) | What does it do? Business flows, user journeys, domain concepts, owned capabilities. | anyone designing a feature or writing stories — PM-readable |
| `system.md` (e.g. `.docs/system.md`) | How is it built and what does it expose? Layers, data flow, domain models, invariants, file index, API surface, outbound dependencies, event contracts. | developers, engineers, and AI agents touching code or integrating with it |
| `service-manifest.md` (`.docs/service-manifest.md`) | Who is this service and what contracts does it publish/consume? Portable registry entry for cross-repo orchestration. | a multi-repo orchestrator skill/agent aggregating many repos |

`docs_context` is the config key and its target file *is* the business file — a short link at
the top of it points to `system.md`. There is no directory to create.

The exact markdown for each of these lives in its own template file under [templates/](templates/)
— see Step 3.

**This skill produces the knowledge layer, not a plan.** It documents what exists and what
contracts a service honors. Deciding *what to build next* in this repo, or in a repo that depends
on one of these contracts, is `feature` (plan) and `implement` (build) — they read `CONTEXT.md`,
`system.md`, and any dependency's `service-manifest.md` as their starting context.

## When to trigger

Use this skill when the user:
- asks to map, document, or refresh the overall system architecture
- wants the `docs_context` layer created or reconciled with the current codebase
- asks for a broad structural overview spanning the whole codebase, not a single feature's diff

## Tool Availability

This skill **reads** graph-based code analysis tools when available. It does not build the
indexes itself — Step 2a first invokes the `codebase-indexing` skill (the single writer) to
ensure they're fresh.

| Tool | Used For | Fallback |
|---|---|---|
| **codebase-memory-mcp** | Layers, file index, data flow tracing | Manual grep + file reading |
| **graphify** | Domain models (god nodes), business workflows, semantic relationships | Manual code reading + docs analysis |

**If neither tool is available**: `codebase-indexing` reports so; proceed with manual analysis (see Step 2 fallback instructions).

---

## Step 1 — Detect framework, source root, context type, and layer state

**Framework** (check project root):
- **Spring Boot** (Java): `pom.xml`/`build.gradle` contains `spring-boot-starter` → source root = deepest common package (e.g. `src/main/java/com/example/service/`)
- **Rails** (Ruby): `Gemfile` contains `rails` → source root = `app/`
- **Nuxt 3** (JS/TS): `nuxt.config.ts`/`.js` → source root = `server/` (BFF) or `pages/` (frontend)
- **Unknown**: degrade gracefully — still produce both files; omit `system.md`'s interface sections (API Surface, External Dependencies, Events) if no HTTP/messaging/outbound surface is found.

**Scope: one repo per run, multi-repo-ready output.** This skill always scans the **current
repo** as a single service. It never scans sibling repos. Every run produces three artifacts:
`CONTEXT.md` (business), `system.md` (technical), and a **service manifest** — the portable,
agent-ready unit a multi-repo orchestrator later aggregates (Step 3.4). The orchestrator node
(platform `context-orchestrator.md`) is **assembled from many repos' manifests by an orchestrator
skill/agent — not produced here.**

**Service count within this repo** (almost always one):
- **Single service** (default): one build file / one deployable → one `service` name, one set of
  the three artifacts. Use `context.md`.
- **Repo genuinely containing multiple deployables** (rare — a true in-repo monorepo with
  `services/*/` each having its own build file): emit the three artifacts *per service directory*,
  each manifest self-identifying with the same `repo.remote` but a distinct `service`. Still no
  cross-repo scanning.

`context-orchestrator.md` is documented for the aggregating orchestrator; a normal per-repo run
does not write it.

**Layer state** — check for `docs_context` (default: `.docs/CONTEXT.md`) and its sibling `system.md`:
- **Absent → fresh generation.** Produce both files from the fact set (Step 2).
- **Present → reconcile.** Read each existing file fully. This is a reconcile, not a rewrite: keep sections that still match reality, update drifted ones, add sections for new structure, and **preserve hand-written prose** by folding it into the nearest matching section. Treat each template's line-cap and required sections as authoritative.
- **Legacy directory layer** (`.context/business.md` + `.context/system.md`, or the older `.context/business.md` + `interface.md` + `implementation.md`): migrate — write the merged content to the current `docs_context` file and its sibling `system.md`, then delete the old `.context/` directory. Report the migration explicitly.
- **Legacy monolithic file** (an existing `.docs/CONTEXT.md` or `.docs/ARCHITECTURE.md` with everything in one file, no companion `system.md`): migrate — split its content into the business sections (→ `docs_context`) and technical/interface sections (→ new `system.md`), per each template. Report the migration explicitly.

Determine the `service` name (kebab-case) from the build file / directory name — used in every file's frontmatter.

---

## Step 2 — Extract Architecture Lessons & Build Service Registry (Tool-Assisted)

### Step 2a — Ensure indexes are fresh, then read lessons

**First, invoke the `codebase-indexing` skill** to build/refresh both the codebase-memory-mcp
structural index and the graphify semantic graph (`.docs/indexing/graphify/LESSONS.md`). This skill is a *reader* —
it does not build indexes itself; `codebase-indexing` is the single writer and is cheap on a
warm repo. This is the one place `architecture` blocks on a full build, because a broad scan
needs fresh indexes.

Then read the artifacts (for both single-service and multi-service contexts):

1. **If `.docs/indexing/graphify/LESSONS.md` was produced by graphify**, parse it to extract:
   - **God nodes** → central entities (identify service boundaries)
   - **Community clusters** → semantically related components (identify owned domains)
   - **Architecture patterns** → recurring structural motifs (extract topology rules). graphify reports that a motif *recurs*; naming it is Step 2b's **Patterns in use** row, via `design-patterns`
   - **Prior decisions** → implicit or explicit ADRs (list in Architecture Decisions section)

2. **If graphify was unavailable** (no `.docs/indexing/graphify/LESSONS.md`): proceed with manual extraction from README and code comments.

**Reuse these facts for both CONTEXT.md and system.md.**

---

### Step 2b — Mine the codebase once (Tool-Assisted)

Do this once and reuse the fact set for CONTEXT.md and system.md. There is no separate
fact-gathering pass for the manifest — it's projected from these same facts in Step 3.3, so
nothing is mined twice. For each signal, try the primary tool first; fall back if unavailable.

| Signal | Primary Tool | Query | Fallback |
|---|---|---|---|
| **Layers + File Index** | `codebase-memory-mcp` `get_architecture()` | Returns layer structure directly | `ls` top-level dirs under source root |
| **Tech stack + entry points** | Manual inspection | `build.gradle`/`pom.xml`/`package.json`, run scripts | grep for common patterns |
| **Domain models** | `graphify` god nodes | Query for entity/model nodes + relationships | grep `entity/**`, `app/models/**` + inspect source |
| **Data flow** | `codebase-memory-mcp` `trace_path(mode=data_flow)` | Trace 2–3 representative chains | Manual call-chain following via `analyze-code` |
| **Endpoints** | `codebase-memory-mcp` `search_graph()` | Find all @GetMapping/@PostMapping/routes | grep `@GetMapping`/`routes.rb`/`server/api/**` |
| **Outbound deps** | `codebase-memory-mcp` `search_graph()` | Find *Client/*Service classes + HTTP calls | grep `*Client`, `*Service`, `WebClient` |
| **Events** | `codebase-memory-mcp` `search_code()` | Search for @RabbitListener/@KafkaListener | grep listeners + `application.yml`/`application.properties` |
| **Business workflows** | `graphify` community detection | Clusters of related entities/endpoints | Read integration tests + `README.md` + docs |
| **Purpose + concepts** | `graphify` semantic relationships | Query entity glossary + domain clusters | Read `README.md`, `docs/`, comments |
| **Patterns in use** | `design-patterns` (`identify` mode) | Pass the facts already mined above — layers, domain models, class names, file index | Same skill, reading the file index manually |

**If both codebase-memory-mcp and graphify unavailable:**
Use `locate-code` / `analyze-code` skills for traversal, or fall back to manual grep + file reading per the "Fallback" column above.

---

## Step 3 — Choose Template & Render (or reconcile) each file

Render **three artifacts per service** (almost always just one service — the current repo):
`CONTEXT.md`, `system.md`, and `service-manifest.md`. The manifest is what makes the output
multi-repo-ready.

### Step 3.1 — Render Service CONTEXT.md

1. Open [templates/context.md](templates/context.md)
2. Fill Service Purpose (what this service does, what domain it owns)
3. Fill Owned Domain (what data/responsibility does it own)
4. Fill User Journeys from Step 2c facts (endpoint groups + listener classes + tests)
5. Fill Domain Concepts (business entities from graphify god nodes)
6. Fill Owned Capabilities from Step 2c

File: `.docs/CONTEXT.md` · ≤150 lines · **`type: business-context`**.

### Step 3.2 — Render system.md

1. Open [templates/system.md](templates/system.md)
2. Fill all required sections from Step 2c facts (layers, data flow, API surface, etc.)
3. Fill **Patterns in Use** from the `design-patterns` `identify` output — only patterns the
   codebase is organized around, capped at 8 bullets. This section records conventions a
   contributor must match; it is not an assessment, so carry no recommendations into it
4. Omit skippable sections (Patterns in Use, API Surface, External Dependencies, Events) if none found

File: `.docs/system.md` · ≤300 lines · **`type: system-context`**.

### Step 3.3 — Project the Service Manifest (do not re-author)

The manifest is **never independently authored** — it's a mechanical extract of the system.md
you just wrote in Step 3.2, plus git identity. This is what keeps three files in sync instead of
three sources of truth: system.md is authored/reconciled, the manifest is *derived* from it.

| Manifest field | Source |
|---|---|
| `repo.remote` / `repo.branch` | `git remote get-url origin` + current default branch (the one fact not already in system.md) |
| `publishes.api` | system.md's **API Surface** section, one entry per row |
| `publishes.events` | system.md's **Events → Outbound (produced)** table |
| `consumes` (events) | system.md's **Events → Inbound (consumed)** table |
| `consumes` (calls) | system.md's **External Dependencies** section — reference the target **by service name**, resolved from the system name / base-URL env var; mark `unknown` if it can't be inferred |
| `owned_domain` | system.md's **Key Domain Models** (condensed to one line) |
| `constraints` | system.md's **Key Invariants** + any auth/SLA/compliance facts already surfaced there |
| `docs.*` | fixed paths — `CONTEXT.md`, `system.md`, `.docs/indexing/graphify/LESSONS.md` |

Steps:
1. Open [templates/service-manifest.md](templates/service-manifest.md)
2. Copy the fields above straight out of the system.md just rendered — do not re-query tools or
   re-derive facts already captured there
3. **Exception — preserve hand-added `consumes` entries.** A dependency reached through a shared
   library or config (no direct HTTP client to grep) won't show up in system.md. If a prior
   manifest has such an entry, carry it forward; this is the one place manual authorship survives
   re-projection

File: `.docs/service-manifest.md` · ≤150 lines · **`type: service-manifest`**. This is the only
artifact designed to be read *out of repo context* — keep it self-contained; pointers only, never
inlined schemas.

**Maintenance:** CONTEXT.md and system.md are the two authored/reconciled docs — they're what
drift over time and what Step 1's reconcile logic applies to. The manifest has no independent
drift: re-projecting it after any system.md change (same `codebase-indexing` `detect_changes`
signal that triggers a reconcile) keeps it correct by construction. If system.md hasn't changed,
skip re-projecting the manifest too.

### Step 3.4 — Orchestrator node (NOT produced by a per-repo run)

The platform `context-orchestrator.md` (Service Registry + topology across services) is assembled
by a separate orchestrator skill/agent that globs `**/.docs/service-manifest.md` across many repo
checkouts and joins their frontmatter — see [templates/context-orchestrator.md](templates/context-orchestrator.md)
and the manifest template's "How the orchestrator uses it". A single-repo `architecture` run
**does not** write it; it only contributes this repo's manifest.

### Rules that apply across all files:
- Every file starts with frontmatter (line 1 = `---`) carrying
  `type / service / version / updated / tags`; set `updated:` to today. On reconcile, bump
  `updated:` only for files whose content actually changed.
- `system.md` has three independently-skippable interface sections (API Surface, External
  Dependencies, Events) plus a skippable **Patterns in Use** — omit a section the service
  genuinely has none of; the rest of the file is always produced. See the template's authoring
  notes for the per-section rules.
- Templates use `[docs_context]` as a placeholder to resolve to the actual configured path when writing the file.

---

## Step 4 — Self-verify (definition of done)

Inline checks — no external linter required:

**All files:**
- **Line caps**: `wc -l` per file ≤ its template's limit:
  - Orchestrator CONTEXT.md: ≤150 lines
  - Single-service CONTEXT.md: ≤150 lines
  - system.md: ≤300 lines
  - service-manifest.md: ≤150 lines
- **Required headings**: grep each `##`/`###` heading the corresponding template requires.
- **Frontmatter**: line 1 is `---`; `type/service/version/updated/tags` all present; `updated` = today.
- **No leftovers**: `grep -n 'TODO\|FIXME\|<!-- fill in'` returns nothing.
- **No legacy leftovers**: no `.context/` directory remains; no monolithic file without a `system.md` sibling.

**system.md only:**
- **File Index paths**: every `src/...` glob resolves on disk (`ls`/glob).

**service-manifest.md:**
- Frontmatter parses as valid YAML; `type: service-manifest`; `service` matches `CONTEXT.md` / `system.md`.
- `repo.remote` and `repo.branch` are filled (not placeholders).
- **Every `publishes` entry traces to a row in this run's system.md** (API Surface / Events) — no field invented independently of system.md.
- `consumes` entries are summaries + pointers — **no inlined schemas or source**; every `consumes[].service` names a target (or is explicitly `unknown`).
- Hand-added `consumes` entries from a prior manifest (Step 3.3's exception) survived the re-projection.
- `docs:` pointers resolve on disk (skip `lessons` if graphify was unavailable).

**LESSONS.md (if generated):**
- File exists at `.docs/indexing/graphify/LESSONS.md`
- Contains god nodes, communities, patterns, decisions sections

**Context dictionary consistency (flag only, don't edit):**
- If `docs_dictionary_file` exists, check its `## Core Context` rows still point at the artifacts
  this run actually wrote (`docs_context`, `system_context`, `service_manifest`). That section is
  owned by `central-workspace` — **never edit it here.** If a row is missing, stale, or points at a
  path this run didn't produce, report it under `Context dictionary may be stale:` and recommend
  re-running `central-workspace`.

Fix and re-check until all pass. Only then report done.

---

## Step 5 — Check README accuracy (flag only)

Compare `README.md`'s claims against what the scan found:
- capabilities/integrations it mentions that no longer exist
- significant capabilities the scan found that it omits

List mismatches under a `README.md may be stale:` heading as suggestions. **Never edit `README.md`** — it's human-voiced prose.

---

## Report

State fresh-generation vs reconcile. List each file as **created / reconciled (what changed) / unchanged / skipped (why)**. Confirm Step 4 self-verification passed. Append the `README.md may be stale:` list if Step 5 found anything.

**Files to report on** (per service — normally one, the current repo):
- `.docs/CONTEXT.md` — created/reconciled/unchanged
- `.docs/system.md` — created/reconciled/unchanged
- `.docs/service-manifest.md` — projected/reprojected/unchanged (skipped if system.md didn't change) ← the multi-repo aggregation unit
- `.docs/indexing/graphify/LESSONS.md` — created/skipped (why) — written by `codebase-indexing`

**Also report:**
- Service(s) documented in this repo, and confirmation the manifest is orchestrator-ready
  (`repo.remote` set, `publishes`/`consumes` traced to system.md, any hand-added `consumes` preserved)
- Tools used: `codebase-indexing` result, codebase-memory-mcp, graphify, manual fallbacks
- Any migrations performed (legacy context → new structure)

Note: this skill produces the **knowledge layer** for this repo, not a build plan — see the intro.
The platform orchestrator node (`context-orchestrator.md`) is **not** produced here either — it is
assembled from this and other repos' manifests by a separate orchestrator run.

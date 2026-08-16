# Template: `docs_context`/system.md

≤300 lines · 8 required sections + 1 skippable pattern section + 3 skippable interface sections · every File Index path
must resolve on disk. This is the single technical file — how the service is built **and**
what it exposes, calls, and consumes/produces.

```markdown
---
type: system-context
service: [service]
version: 1.0
updated: [today]
tags: [architecture, technical, interface, load-first]
---

# [service] — System Context

> For developers, engineers, and AI agents. Read this before reading source code.
> Business-facing companion: [CONTEXT.md](CONTEXT.md).

## Service Identity
[1–3 lines: what it does, owning team, region/domain]

## Tech Stack
- Runtime: [language+version, framework+version]
- Build: [tool] · DB: [db + migrations] · Cache/Lock: [if any]
- Messaging: [broker + direction] · API format: [REST/GraphQL, content type]
- External SDKs: [key SDKs] · Testing: [frameworks]

## Entry Points
```bash
[exact build / test / format / run commands]
```

## Architecture Map
| Layer | Responsibility | Key Classes |
|---|---|---|
[one row per top-level package/dir]

## Data Flow
[trigger → component → component → outcome, one line per representative flow]

## Patterns in Use
Structural conventions a contributor must match, not a catalog of every pattern present.
Max 8 bullets — omit the section if the codebase carries no load-bearing pattern.
Format: `**[Pattern]** — [where] · [the signal that identified it]`
- **[Pattern]** — [package or class] · [signal]

## API Surface
[version note]. Format: `METHOD /path — purpose`. No auth/field details.

### [Resource Group]
- `METHOD /path` — one-line purpose

## External Dependencies
Outbound calls grouped by target system.

### [System]
> Purpose: [one line] · Client: [class] · Base URL via `[ENV_VAR]`
> (or: "Accessed via [SDK], not raw HTTP.")

| Method | Path | Purpose |
|---|---|---|
| GET | /path | one-line purpose |

## Events
> Exchange: [name if applicable]

### Inbound (consumed)
| Event | Queue | Routing Key | Notes |
|---|---|---|---|

### Outbound (produced)
| Event | Routing Key | Trigger |
|---|---|---|

## Key Domain Models
- **[Model]** — [relationships + responsibilities]

## Key Invariants
- [non-obvious constraint a senior dev would miss reading code cold — max 5 bullets]

## File Index
```
[layer]: [glob]/**
```
```

**Authoring notes:**
- Required sections (always present): Service Identity, Tech Stack, Entry Points,
  Architecture Map, Data Flow, Key Domain Models, Key Invariants, File Index.
- Skippable sections: **API Surface** (omit if no HTTP surface), **External Dependencies**
  (omit if no outbound calls), **Events** (omit if no event bus). Omit the whole section —
  never leave an empty table.
- Record the exact `##`/`###` headings you use — a reconcile pass must match them.
- Key Invariants is the one section that needs judgment (signing, ack semantics, counters,
  coverage floors). Derive it from what the code actually enforces — never leave a
  `<!-- fill in -->` placeholder behind.
- **Migrating from an older layer:**
  - Three-file layer (`interface.md` + `implementation.md`): former `implementation.md`
    content maps to the required sections; former `interface.md` content maps to API Surface /
    External Dependencies / Events. The old `## External Systems` table merges into
    `## External Dependencies` (system-level purpose line + endpoint table per system).
  - Directory layer (`.context/business.md` + `.context/system.md`): move `system.md` to sit
    beside the new `docs_context` file (default `.docs/system.md`) — content is unchanged.
  - Monolithic single-file layer (`.docs/CONTEXT.md` or `.docs/ARCHITECTURE.md` with everything
    in one file): split it — technical content (stack, layers, data flow, models, invariants,
    API/dependencies/events) moves here; business content moves to `docs_context`'s `context.md`
    template.

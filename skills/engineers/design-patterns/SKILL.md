---
name: design-patterns
description: Identify, choose, apply, and audit design patterns — names the patterns a codebase already uses, evaluates fit before recommending new ones, applies them incrementally during implementation, and reviews code or a design doc for opportunities and anti-patterns
metadata:
  phase: "orient | plan | implement | validate"
  input: "a codebase to identify patterns in, a design option, an implementation phase, or a review scope (path, spec, or diff)"
  output: "named patterns in use — or pattern recommendations with rationale and trade-offs, or a pattern review report"
  dependencies: "onboard-project, find-patterns"
---

# Design Patterns

Pick the pattern that fits the problem, apply it without breaking behavior, and say plainly when no pattern is warranted. Covers the GoF catalog plus modern (Repository, CQRS, Circuit Breaker) and architectural (Hexagonal, Clean, Event-Driven) patterns.

## When to trigger

Use this skill when the user:
- asks which pattern fits a problem, or names a pattern to apply (Strategy, Factory, Observer, …)
- wants existing code reviewed for pattern opportunities, misapplied patterns, or anti-patterns
- describes a structural pain — "decouple this", "make this extensible", "reduce coupling", "this switch keeps growing"
- mentions Refactoring Guru, Gang of Four, or GoF patterns

It is also invoked by:
- `feature` — during design, to check whether a chosen option needs a pattern and to record it under **Conventions to Follow**
- `implement` — per phase, when the **Code quality rules** call for a service or a thin model and the shape of that service is an open question

Do **not** use it for general code comprehension — that's `analyze-code`. Do not use it to find existing examples in this codebase — that's `find-patterns`.

## Variables

`mode`: `identify` | `design` | `apply` | `review` — inferred in Step 2 unless the caller states it
`scope`: the spec file, phase, path, or diff range under consideration

---

## Step 1 — Load Project Context

1. If this is a new session and project context is not yet loaded, invoke the `onboard-project` skill before continuing.
2. Load context per `# WORKSPACE` → **Context Loading**, matching on the files or keywords in `scope`. Documented conventions and ADRs outrank any pattern in this catalog — a pattern that contradicts a recorded architectural decision is not a recommendation, it's a conflict to surface.
3. Invoke `find-patterns` before recommending anything structural. A pattern already used in this codebase beats a textbook-correct one that appears nowhere else — consistency is worth more than catalog purity.

---

## Step 2 — Pick the Mode

| The caller has… | Mode | Go to |
|---|---|---|
| A codebase and wants to know what patterns are already in it | `identify` | Step 3.5 |
| A design option or spec, no code yet | `design` | `modes/design.md` |
| An approved phase to build, seams being decided now | `apply` | `modes/apply.md` |
| Existing code, a path, or a diff to audit | `review` | `modes/review.md` |

If both a design doc and code exist and the user wants both assessed, run `design` then `review` — do not merge them into one pass.

`identify` is the cheap mode and the only one that skips Step 3b — nothing is being proposed, so
there is nothing to disqualify. It reports what exists and stops, without ever leaving this file.
`design`, `apply`, and `review` each run `SKILL.md` Step 3 as their first substep, then live in
their own file under `modes/` — read only the one the table above names. This keeps a
single-mode invocation from paying for the other two modes' instructions.

---

## Step 3 — Identify Candidates and Test Their Fit

`design`, `apply`, and `review` pass through this step; `identify` skips straight to Step 3.5. Two parts: find candidates, then try to disqualify them.

### 3a. Map smells to candidate patterns

| Code smell | Candidate pattern(s) |
|---|---|
| Growing `if/else` or `switch` on a type or flag | **Strategy**, **State**, **Command** |
| Copy-pasted logic differing in one or two steps | **Template Method**, **Strategy** |
| Constructor or function with 4+ parameters, many optional | **Builder** |
| `new ClassName()` scattered; adding a type means editing 5 files | **Factory Method**, **Abstract Factory** |
| God class or oversized file doing several jobs | **Facade**, **Mediator**, **Command** |
| Objects holding references to many others just to notify them | **Observer**, **Mediator**, **Pub/Sub** |
| Adding a feature requires modifying existing classes | **Decorator**, **Strategy**, **Visitor** |
| Cross-cutting concerns (logging, auth, caching, retries) tangled in business logic | **Decorator**, **Proxy** |
| Undo/redo, audit history, or replay needed | **Command**, **Memento**, **Event Sourcing** |
| Traversal or `if leaf else branch` checks over a tree | **Iterator**, **Composite**, **Visitor** |
| Incompatible third-party interface wrapped ad hoc at each call site | **Adapter** |
| Class explosion across two independent dimensions | **Bridge** |
| Thousands of similar objects under memory pressure | **Flyweight**, **Prototype** |
| SQL/ORM calls scattered through services; tests need a real database | **Repository** |
| `new Dependency()` deep inside business logic; untestable without real services | **Dependency Injection** |
| Retry loops without backoff; one slow dependency cascading into everything | **Circuit Breaker**, **Retry with Backoff** |
| Multi-service workflow with no rollback story | **Saga** |
| Domain code importing HTTP/DB packages directly | **Hexagonal**, **Clean Architecture** |
| Long sequential transform where each stage calls the next | **Pipe and Filter** |

### Loading pattern detail

**Check context before reading.** If `catalog.md`, or a pattern's `## <Pattern>` section, was
already read earlier in this run — a prior mode pass on the same task, an earlier candidate that
shares a file, an earlier `identify` call on another service — reuse it. Re-reading a file already
in context spends tokens for nothing.

Read `references/catalog.md` — one line per pattern, ~1k tokens, sized to be read whole. It maps
every pattern to its recognition signal and its detail location.

Then read **only the `## <Pattern>` section** of the file it names, for the two or three
candidates that survived. Never read a whole detail file to reach one pattern: each holds 5–11
patterns, so a whole-file read costs roughly 9× the section you need.

Each core pattern section carries intent, the code smells it solves, when *not* to apply, and
language-idiomatic alternatives — enough to run Step 3b and to write a `design` or `review`
recommendation. Structure diagrams, refactoring steps, and before/after code live in a separate
`-examples.md` file per category, with matching `## <Pattern>` headers. **Only `apply` mode loads
it** (see `modes/apply.md`) — `design` never names classes or methods, and `review` only
recommends, so the example tier is dead weight for both.

| File | Covers |
|---|---|
| `references/catalog.md` | all patterns, one line each — **the entry point** |
| `references/creational-patterns.md` | object creation — core |
| `references/structural-patterns.md` | composition and coupling — core |
| `references/behavioral-patterns.md` | interaction and responsibility — core |
| `references/modern-patterns.md` | data access, resilience, messaging — core, no example tier |
| `references/architectural-patterns.md` | layering and system structure — core, no example tier |
| `references/creational-examples.md`, `structural-examples.md`, `behavioral-examples.md` | structure diagrams, refactoring steps, before/after code — **`apply` mode only** |

### 3b. Try to disqualify each candidate

A candidate survives only if every answer holds:

1. **Problem match** — does it solve the actual problem, or merely resemble it? Read the reference's **When NOT to apply** before answering.
2. **Complexity trade-off** — is the result simpler than what's there now? A pattern that adds more files than the problem warrants is over-engineering.
3. **Language fit** — is there a native idiom that gets the same result with less machinery? First-class functions, decorators, channels, traits, `@dataclass`, functional options. Prefer the idiom.
4. **Codebase fit** — does this codebase already solve this problem another way (per `find-patterns`)? If so, follow that instead, or make the case for changing it everywhere.
5. **Scale** — is there a second variant today, or only a hypothetical one? One implementation needs no abstraction.

**Report disqualified candidates too.** "Considered Strategy, rejected — only one algorithm exists and the language has first-class functions" is a finding, not a gap.

---

## Step 3.5 — Mode: Identify (naming what already exists)

Name the patterns a codebase already uses. Recommends nothing, changes nothing. This is what
`architecture` invokes during a system scan, so it is built to stay cheap.

1. **Read `references/catalog.md` and nothing else** — its **Recognize by** column is the whole
   working set. Do not preload detail files; a run that names ten patterns should still have read
   one reference file. If it's already in context — from an earlier step this run, or an earlier
   `identify` call on another service in the same scan — skip the read and reuse it.
2. **Match against structure you already have.** Prefer signals from the caller's existing fact
   set — a graph's architecture map, class and directory names, an existing file index — over
   fresh file reads. When invoked by `architecture`, the mined facts from its Step 2 are the
   input; do not re-scan the repo.
3. **Confirm before naming.** A fingerprint match is a hypothesis (see the catalog's closing
   section). Read the pattern's `## <Pattern>` **Intent** paragraph only when a match is load-bearing
   or contested — not for every hit.
4. **Report one line per pattern:**

```
- **<Pattern>** (<category>) — `path/or/package` · <the signal that identified it> · confidence: HIGH | MEDIUM | LOW
```

Rules that keep this mode honest and cheap:

- **Name what carries weight.** A pattern used once incidentally is noise; a pattern the codebase
  is organized around is a fact worth recording.
- **Report unnamed patterns by their pattern name.** A map of closures is Factory Method whether or
  not the word appears anywhere — that is the normal case in Go, Python, Rust, and TypeScript.
- **Never upgrade identification into critique.** "Singleton, 4 call sites, HIGH" is in scope;
  "Singleton, should be DI" is `review` mode. If the caller wants an assessment, say that `review`
  provides it and stop.

---

## Report

`identify` ends with its one-line-per-pattern list and nothing else. Every other mode ends with:

1. **Patterns recommended or applied** — one line each, with the problem solved
2. **Patterns considered and rejected** — one line each, with the disqualifying reason
3. **Before/after** — for `apply` mode, the key structural change
4. **Extension point** — how to add the next variant
5. **Open judgment calls** — anything needing a human decision, flagged explicitly

---

## Constraints

1. **Patterns are tools, not goals.** The right answer is often "no pattern — delete the abstraction."
2. **Language idioms first.** A closure, a decorator, a channel, or a trait beats a class hierarchy that reimplements it.
3. **One pattern at a time.** Apply and verify one before starting the next.
4. **Preserve behavior.** Pattern work is a refactor unless the user asked for a behavior change.
5. **Follow the codebase before the catalog.** An established local convention outranks a textbook structure.
6. **Never invent a variant.** Do not add strategies, subclasses, or handlers the request didn't ask for just to justify the pattern.
7. **Review mode does not refactor.** It reports; applying is a separate, explicitly requested pass.

---

## Attribution

Consolidated from two upstream skills:

- [Software Design Patterns](https://mcpmarket.com/tools/skills/software-design-patterns) (mcpmarket) — the identification / application / refactoring workflows and the creational, structural, and behavioral reference catalogs.
- [sirius-zuo/design-pattern-skill](https://github.com/sirius-zuo/design-pattern-skill) — MIT License, Copyright (c) 2026 sirius-zuo — the design-doc and code review modes, the modern and architectural catalogs, and the review report template.

The fit test (Step 3b), the mode routing, and the pipeline integration with `feature` and `implement` are additions made during consolidation, not upstream content.

# design-patterns

> Choose, apply, and audit design patterns — with a fit test that lets the answer be "no pattern".

---

## What it does

`design-patterns` carries one catalog (GoF + modern + architectural, 38 patterns) and one decision procedure, used in four modes:

- **Identify** — name the patterns a codebase already uses. Reports what exists, recommends nothing. This is what `architecture` calls during a system scan, and it's the cheap mode: one ~1.4k-token catalog read, no detail files.
- **Design** — assess a spec or design option before code exists. Surviving patterns come back as entries for the spec's **Conventions to Follow**, phrased as rules to validate against, never as class names.
- **Apply** — apply one pattern to code being written, one role at a time, tests green between each step.
- **Review** — audit existing code or a design doc for patterns in use, opportunities, and anti-patterns, and write a report. Review mode recommends only; it never refactors.

The three proposing modes pass through the same two-part gate: map the smell to candidate patterns, then try to **disqualify** each candidate on problem match, complexity, language idiom, codebase fit, and scale. Rejected candidates are reported alongside accepted ones — "considered Strategy, rejected: one algorithm, and the language has first-class functions" is a finding, not a gap. `identify` skips the gate: nothing is being proposed, so there is nothing to disqualify.

### Three-tier reference, plus a mode-file split

Pattern knowledge is split so a run loads roughly what it uses. `catalog.md` is the dictionary — every pattern in one line, read whole. The detail files are read **by section**, never whole, and split again into a core tier (needed by every proposing mode) and an example tier (needed only by `apply`):

| File | ~Tokens | Covers |
|---|---|---|
| `catalog.md` | 1,400 | all 38 patterns, one line each — recognition signal + where the detail lives |
| `creational-patterns.md` (core) | 1,600 | Factory Method, Abstract Factory, Builder, Prototype, Singleton — intent, applicability, idioms |
| `structural-patterns.md` (core) | 1,600 | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy — intent, applicability, idioms |
| `behavioral-patterns.md` (core) | 2,900 | Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor — intent, applicability, idioms |
| `modern-patterns.md` | 1,700 | Repository, Dependency Injection, Circuit Breaker, Event Sourcing, CQRS, Saga, Retry with Backoff, Pub/Sub — no example tier |
| `architectural-patterns.md` | 1,700 | MVC/MVP/MVVM, Hexagonal, Clean Architecture, Layered, Microservices, Event-Driven, Pipe and Filter — no example tier |
| `creational-examples.md` / `structural-examples.md` / `behavioral-examples.md` | 1,300 / 1,600 / 2,500 | structure diagrams, refactoring steps, before/after code — **`apply` mode only** |

The core/example split exists because `design` and `review` never name classes or methods — a structure diagram and a before/after code sample are dead weight for a mode that only recommends. Only `apply` loads the matching `-examples.md` section, for the one pattern it's actually transforming. Modern and architectural patterns carry no example tier — they read as prose, not as a code shape to copy.

The same idea applies one level up: `SKILL.md`'s own `design`/`apply`/`review` bodies live in `modes/design.md`, `modes/apply.md`, `modes/review.md`, loaded only after Step 2 resolves the mode. A single-mode invocation no longer pays for the other two modes' instructions — `identify`, the mode `architecture` calls once per service in a scan, never leaves `SKILL.md` at all.

Each core pattern entry carries intent, the code smells it solves, **when not to apply**, and language-idiomatic alternatives for Go, Python, Java, Rust, and TypeScript — because the idiom usually beats the classic form. The matching example entry, when one exists, carries the structure diagram, refactoring steps, and a before/after code sample.

---

## When to use

- During `architecture` — names the patterns in use for `system.md`'s **Patterns in Use** section, from facts the scan already mined (no second pass over the repo)
- During `feature` planning — check whether a design option needs a pattern, and record it as a convention
- During `implement` — when the **Code quality rules** call for a service or a thin model and the shape of that service is open
- Standalone — "review this codebase for pattern opportunities", "which pattern fits this?", "refactor this switch to Strategy"

Not for general code comprehension (`analyze-code`) or for finding existing examples in this repo (`find-patterns` — which this skill invokes first, since a convention already in the codebase outranks a textbook-correct pattern that appears nowhere else).

---

## Install

```bash
npx skills add pdkproitf/skills@design-patterns
```

---

## Usage

**Claude Code:**
```
/design-patterns review src/billing/
/design-patterns this payment switch keeps growing
/design-patterns validate docs/specs/1711234567-feature-multi-provider-payouts.md
```

**Other tools:**
```
@design-patterns <a design option, an implementation phase, or a review scope>
```

---

## Output

**Identify mode** — one line per pattern, nothing else:

```
- **Strategy** (behavioral) — `billing/providers/` · injected single-method interface, 4 impls · confidence: HIGH
- **Repository** (modern) — `internal/store/` · *Repository interfaces owning all queries · confidence: HIGH
- **Circuit Breaker** (modern) — `pkg/httpx/` · gobreaker around outbound calls · confidence: MEDIUM
```

**Design and apply modes** — recommendations inline:

```
### Pattern: Strategy — payment provider selection

**Problem:** provider choice is a switch that grows with every new provider
**Why this fits:** each branch is already a self-contained algorithm; Go interfaces make this a
                  one-method interface, no class hierarchy
**Trade-offs:** one file per provider; the registry becomes the place to look for "what exists"
**Confidence:** HIGH

Considered and rejected:
- Abstract Factory — only one product type is created; Factory Method territory at most
```

**Review mode** — a structured report written to `{review_dir}/YYYY-MM-DD-<project>-pattern-review.md` (default `docs/review/`) and echoed inline, covering patterns in use, recommended patterns with impact and priority, candidates considered and rejected, anti-patterns observed, and a machine-parsable JSON summary.

---

## Attribution

This skill consolidates two upstream skills that overlapped heavily on the GoF catalog but diverged on everything around it:

| Source | What came from it |
|---|---|
| [Software Design Patterns](https://mcpmarket.com/tools/skills/software-design-patterns) (mcpmarket) | Identification, application, and refactoring workflows; creational, structural, and behavioral catalogs with intent, structure diagrams, refactoring steps, and before/after examples |
| [sirius-zuo/design-pattern-skill](https://github.com/sirius-zuo/design-pattern-skill) — MIT, © 2026 sirius-zuo | Design-doc and code review modes; modern and architectural catalogs; per-pattern *when not to apply* and language considerations; the review report template |

The fit test, the mode routing, and the integration with `feature` and `implement` were added during consolidation.

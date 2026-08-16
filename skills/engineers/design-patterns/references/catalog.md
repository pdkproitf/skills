# Pattern Catalog

The dictionary. One line per pattern: what it looks like in code, and where its detail lives.

**Load this file first, and often only this file.** It is sized to be read whole (~1k tokens).
The detail files are not — each holds 5–11 patterns, so reading one to reach a single pattern
costs roughly 9× what that pattern's section costs. When you need detail, read **only the
`## <Pattern>` section** of the file named in the last column.

**Recognize by** is a structural fingerprint — what you would see in a file listing, a class
name, or a call site. It identifies a pattern that is *already there*. To go the other way —
from a problem to a candidate pattern — use the smell table in `SKILL.md` Step 3a.

---

## Creational — `creational-patterns.md`

| Pattern | Recognize by |
|---|---|
| **Factory Method** | `create*`/`New*`/`make*` returning an interface; a registry map of constructors |
| **Abstract Factory** | One factory type with several `create*` methods returning a matched family |
| **Builder** | Chained setters returning `self`/`this`, terminated by `build()`; functional options |
| **Prototype** | `clone()`/`copy()`/`With*` producing a variant of an existing instance |
| **Singleton** | `getInstance()`, `sync.Once`, `lazy_static!`, module-level shared instance |

## Structural — `structural-patterns.md`

| Pattern | Recognize by |
|---|---|
| **Adapter** | A type named `*Adapter`/`*Wrapper` implementing our interface over a foreign one |
| **Bridge** | An abstraction holding an implementor interface, both subclassed independently |
| **Composite** | A node type that both implements and contains a collection of its own interface |
| **Decorator** | A type implementing an interface *and* wrapping an instance of it; middleware chains |
| **Facade** | One coarse service method fronting many subsystem calls; `*Facade`/`*Service` |
| **Flyweight** | An intern pool / shared-instance cache; state passed in rather than stored |
| **Proxy** | Same interface as the real subject, delegating after a check, cache, or lazy init |

## Behavioral — `behavioral-patterns.md`

| Pattern | Recognize by |
|---|---|
| **Chain of Responsibility** | Handlers with a `next` reference; middleware/filter/interceptor stacks |
| **Command** | Action objects with `execute()`/`undo()`; a queue or history of them |
| **Interpreter** | An AST node hierarchy with an `evaluate()`/`interpret()` method per node |
| **Iterator** | A cursor type with `next()`/`hasNext()`; generators; `Iterator` trait impls |
| **Mediator** | A hub every participant references instead of referencing each other; event bus |
| **Memento** | `snapshot()`/`restore()` pairs; an opaque state object handed back to its owner |
| **Observer** | `subscribe`/`addListener`/`notify*`; a collection of callbacks fired on change |
| **State** | One type per state sharing an interface, with a `current`/`state` field swapped |
| **Strategy** | An injected single-method interface, chosen at construction or per call |
| **Template Method** | A base method calling abstract/overridable steps subclasses fill in |
| **Visitor** | `accept(visitor)` on elements; a `visit*` method per element type |

## Modern — `modern-patterns.md`

| Pattern | Recognize by |
|---|---|
| **Repository** | `*Repository` interfaces owning all queries for one aggregate |
| **Dependency Injection** | Constructors taking interfaces; a container, wiring module, or composition root |
| **Circuit Breaker** | Open/half-open/closed state around a remote call; `gobreaker`, Resilience4j |
| **Event Sourcing** | An append-only event store plus projections rebuilt from it; no in-place updates |
| **CQRS** | Separate command and query handlers, often over separate read/write models |
| **Saga** | A multi-service workflow with a compensating action per step; orchestrator or choreography |
| **Retry with Backoff** | Retry helpers with attempt caps and delay growth; `tenacity`, `backoff` |
| **Pub/Sub** | Publish to a topic/exchange with subscribers resolved at runtime, not by reference |

## Architectural — `architectural-patterns.md`

| Pattern | Recognize by |
|---|---|
| **MVC / MVP / MVVM** | Controller/presenter/viewmodel layer between routes and models |
| **Hexagonal (Ports & Adapters)** | A domain package importing no framework; `ports/` + `adapters/` |
| **Clean Architecture** | Entities → use cases → adapters → drivers, dependencies pointing inward only |
| **Layered (N-Tier)** | `presentation`/`business`/`data` (or `controller`/`service`/`repository`) packages |
| **Microservices** | Several independently deployable services with their own builds and datastores |
| **Event-Driven** | Services reacting to broker messages rather than calling each other synchronously |
| **Pipe and Filter** | Composable single-purpose stages chained into a pipeline; iterator/stream chains |

---

## Reading a recognition signal honestly

A fingerprint match is a hypothesis, not a finding. Two failure modes to guard against:

- **Named but not applied** — a class called `PaymentFactory` that is a plain constructor wrapper is not Factory Method. Check that the pattern's *role structure* is present, not just its vocabulary.
- **Applied but not named** — a map of closures keyed by type is Factory Method; a `func(Handler) Handler` chain is Chain of Responsibility. Idiomatic code frequently implements a pattern without ever naming it, and that is the normal case in Go, Python, Rust, and TypeScript.

When a fingerprint matches, confirm against the pattern's **Intent** in its detail section before reporting it.

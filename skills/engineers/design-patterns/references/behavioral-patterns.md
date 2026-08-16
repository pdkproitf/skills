# Behavioral Design Patterns Reference

These patterns are concerned with algorithms and the assignment of responsibilities between
objects. During refactoring, look for complex conditionals, tight event coupling, duplicated
algorithm structures, and objects that know too much about each other.

---

## Chain of Responsibility

**Intent:** Pass a request along a chain of handlers. Each handler decides either to process
the request or to pass it to the next handler in the chain.

**Code smells it solves:**
- Nested `if/else if/else if` chains for request processing
- Single function doing validation, auth, logging, and business logic
- Adding a new processing step requires modifying existing code

**When to apply during refactoring:**
- HTTP middleware stacks (auth → rate limit → validate → handle)
- Input validation chains
- Event processing pipelines
- Log level filtering

**When NOT to apply:**
- The request always has exactly one handler — call it directly
- The chain grows long enough that tracing a request through it becomes the new problem

---

## Command

**Intent:** Encapsulate a request as an object, thereby letting you parameterize clients
with different requests, queue or log requests, and support undoable operations.

**Code smells it solves:**
- UI callbacks with complex inline logic
- No undo/redo capability in an editor or form
- Operations that need to be queued, scheduled, or replayed
- Transaction logging that duplicates business logic

**When to apply during refactoring:**
- Adding undo/redo to an application
- Implementing a task queue or job scheduler
- Decoupling UI actions from business logic
- Creating macro/batch operations from individual actions

**When NOT to apply:**
- A plain callback or closure suffices — full Command objects add ceremony
- Operations are fire-and-forget and undo is not needed

---

## Interpreter

**Intent:** Define a representation for a simple language's grammar, plus an interpreter that
uses the representation to evaluate sentences in that language.

**Code smells it solves:**
- Configuration or query strings parsed ad hoc with regex and string splitting
- Rule logic hardcoded in conditionals that users keep asking to make configurable
- The same expression-evaluation logic reimplemented in several places

**When to apply during refactoring:**
- Configuration DSLs, filter/query expressions, feature-flag rules, formula evaluators
- The grammar is small, stable, and efficiency is not the primary concern

**When NOT to apply:**
- The grammar is non-trivial — use a parser generator or an existing parsing library
- Evaluation is on a hot path — walking a tree per evaluation is slow

**Language-idiomatic alternatives:**
- **All languages:** recursive-descent parser plus a Visitor over the resulting AST
- **Java:** ANTLR · **Rust:** `pest`, `nom` · **Python:** `lark`, `pyparsing` · **Go:** `participle`

---

## Iterator

**Intent:** Provide a way to access elements of an aggregate object sequentially without
exposing its underlying representation.

**Code smells it solves:**
- Client code depends on the internal structure of a collection
- Same traversal logic duplicated for different collection types
- Complex data structures (trees, graphs) with manual traversal code

**When to apply during refactoring:**
- Unifying traversal across different data structures
- Hiding internal collection representation from consumers
- Supporting multiple traversal strategies (depth-first, breadth-first, filtered)

**When NOT to apply:**
- The language already provides iteration — use it rather than reimplementing the protocol

**Language-idiomatic alternatives (prefer these):**
- **JavaScript/TypeScript:** `Symbol.iterator` protocol, generators (`function*`)
- **Python:** `__iter__` / `__next__` protocol, generators (`yield`)
- **Go:** Range over channels, or iterator functions (Go 1.23+)
- **Rust:** `Iterator` trait (`.iter()`, `.map()`, `.filter()`)
- **Java:** `Iterable<T>` / `Iterator<T>`, Streams API

In modern languages, you rarely need to implement Iterator as a class — use the language's
iteration protocol instead.

---

## Mediator

**Intent:** Define an object that encapsulates how a set of objects interact. Promotes loose
coupling by keeping objects from referring to each other explicitly.

**Code smells it solves:**
- Classes that hold references to many other classes
- Changing one class requires changes in many others
- Complex web of direct communications between objects
- God class that coordinates everything

**When to apply during refactoring:**
- UI components that directly update each other (form field A disables button B, shows panel C)
- Chat rooms, event buses, or notification systems
- Workflow orchestration where steps depend on each other

**When NOT to apply:**
- Only two objects communicate — a direct reference is clearer
- The mediator is becoming a god object — decompose it instead of growing it

**Language-idiomatic alternatives:**
- **JavaScript:** Event emitter / EventTarget
- **React:** Lift state up + context
- **Redux/Zustand/Pinia:** Centralized state management is a mediator

---

## Memento

**Intent:** Capture and externalize an object's internal state so that the object can be
restored to this state later, without violating encapsulation.

**Code smells it solves:**
- Manual state snapshots with exposed internals
- Undo implementations that store full object copies
- State history that breaks encapsulation

**When to apply during refactoring:**
- Adding undo/redo (pairs with Command pattern)
- Implementing save/load for application state
- Checkpoint/rollback for long-running operations
- Form state management (save draft, restore previous values)

**When NOT to apply:**
- Serialization (JSON, protobuf) already covers state capture
- The state is trivial to recreate from scratch

---

## Observer

**Intent:** Define a one-to-many dependency between objects so that when one object changes
state, all its dependents are notified automatically.

**Code smells it solves:**
- Polling for changes (`setInterval(() => checkForUpdates(), 1000)`)
- Manual notification calls scattered after every state change
- Tight coupling between data producers and consumers
- UI not updating when underlying data changes

**When to apply during refactoring:**
- Event systems, pub/sub messaging
- Reactive UI updates (model changes → view updates)
- Plugin/extension systems that react to core events
- Log aggregation, metrics collection

**When NOT to apply:**
- There is exactly one observer and the relationship is fixed — call it directly
- Cascading updates are already hard to trace; more indirection makes debugging worse

**Language-idiomatic alternatives:**
- **JavaScript:** `EventTarget`/`EventEmitter`, RxJS Observables
- **Python:** Signals (Django), `blinker`, or `asyncio` events
- **Go:** Channels
- **Rust:** Channels (`mpsc`), or callback registration
- **React:** `useEffect` with dependencies, Context, state management libraries

---

## State

**Intent:** Allow an object to alter its behavior when its internal state changes.
The object will appear to change its class.

**Code smells it solves:**
- Giant `switch` or `if/else` on a status field in every method
- Boolean flags that control behavior: `if (isEditing && !isLocked && hasPermission)`
- State transitions scattered and error-prone

**When to apply during refactoring:**
- Document workflow (draft → review → approved → published)
- UI component modes (viewing, editing, loading, error)
- Connection states (disconnected, connecting, connected, error)
- Game entity behavior (idle, moving, attacking, dead)

**When NOT to apply:**
- Two states with minimal behavior — a boolean is enough
- The states do not actually behave differently, they only label data

---

## Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them
interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**Code smells it solves:**
- `if/else` or `switch` choosing an algorithm at the start of a function
- Duplicated methods that differ only in the core algorithm step
- Hard to add new algorithms without modifying existing code

**When to apply during refactoring:**
- Sorting with different comparators
- Payment processing with multiple providers
- Validation with different rule sets
- Routing with different strategies (fastest, shortest, cheapest)
- Compression, encryption, or serialization with multiple algorithms

**When NOT to apply:**
- Only one algorithm will ever exist — the abstraction is unnecessary
- The caller always knows the algorithm at compile time — just call it

**Language-idiomatic alternatives:**
- **JavaScript/TypeScript/Python:** First-class functions — just pass a function, no class needed
- **Go:** Function types (`type Strategy func(data) result`)
- **Rust:** Closures or trait objects (`Box<dyn Strategy>`)

---

## Template Method

**Intent:** Define the skeleton of an algorithm in a method, deferring some steps to
subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

**Code smells it solves:**
- Multiple functions with the same overall structure but different details
- Copy-pasted "framework" code with small variations
- Hooks or callbacks embedded in a rigid sequence

**When to apply during refactoring:**
- Data import pipelines (read → validate → transform → save) with different formats
- Test setup/teardown sequences with varying test bodies
- Document generation (header → content → footer) with different content types
- Build processes with language-specific compilation steps

**When NOT to apply:**
- The varying step must change at runtime — prefer Strategy (composition over inheritance)
- The inheritance hierarchy is already deep enough that the algorithm is hard to follow

**Language-idiomatic alternatives:**
- **JavaScript/TypeScript:** Higher-order functions that accept step functions as callbacks
- **Python:** Abstract base classes (`abc.ABC`), or just pass functions
- **Go:** Function parameters or interfaces for the varying steps

---

## Visitor

**Intent:** Represent an operation to be performed on the elements of an object structure.
Visitor lets you define a new operation without changing the classes of the elements.

**Code smells it solves:**
- Adding a new operation requires modifying every element class
- `instanceof` / type switch chains for performing different operations on different types
- Operations on a hierarchy are scattered across the element classes

**When to apply during refactoring:**
- AST processing (type checker, optimizer, code generator as separate visitors)
- Document export (export to PDF, HTML, Markdown without changing document classes)
- Tax/pricing calculations across different product types
- Serialization of heterogeneous object graphs

**When NOT to apply:**
- The set of element types changes often — every new type forces an edit to every visitor
- Only one operation is needed — add a method to the class instead

**Trade-off:** Adding new element types requires updating all visitors. Use Visitor when
operations change frequently but the element types are stable.

**Language-idiomatic alternatives:**
- **TypeScript:** Discriminated unions + exhaustive switch (often simpler)
- **Python:** `functools.singledispatch` or `match` statement (3.10+)
- **Rust:** `enum` + `match` (exhaustive pattern matching)
- **Kotlin:** `sealed class` + `when` expression

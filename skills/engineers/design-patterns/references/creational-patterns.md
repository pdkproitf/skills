# Creational Design Patterns Reference

These patterns provide flexible object creation mechanisms. During refactoring, look for
scattered `new` keywords, complex constructors, and duplicated initialization logic.

---

## Factory Method

**Intent:** Define an interface for creating an object, but let subclasses decide which class
to instantiate.

**Code smells it solves:**
- `if/else` or `switch` statements that decide which class to instantiate
- Scattered `new ClassName()` calls that change when a new type is added
- Constructor logic duplicated across multiple call sites

**When to apply during refactoring:**
- You're adding a new type and realize you have to update conditionals in 5 places
- Tests need to substitute different implementations
- The creation logic is more complex than just calling a constructor

**When NOT to apply:**
- Creation is trivial — a constructor call is clearer than a factory
- Only one concrete type will ever be created

**Language-idiomatic alternatives:**
- **TypeScript/JavaScript:** A simple factory function (no class hierarchy needed)
- **Python:** Class methods as alternative constructors (`@classmethod`)
- **Go:** Constructor functions (`NewXxx()` convention)
- **Rust:** Associated functions and the `From`/`Into` traits

---

## Abstract Factory

**Intent:** Provide an interface for creating families of related objects without specifying
their concrete classes.

**Code smells it solves:**
- Multiple factory methods that always create objects from the same "family"
- Platform-specific code mixed with business logic (`if (platform === 'ios') ...`)
- Inconsistent object families (mixing light theme buttons with dark theme dialogs)

**When to apply during refactoring:**
- You have related objects that must be used together (UI themes, database drivers, OS services)
- Platform or environment-specific creation logic is scattered
- You're introducing multi-tenant or multi-brand support

**When NOT to apply:**
- Only one kind of object is being created — use Factory Method instead
- The family is fixed and will not grow — the extra indirection buys nothing

**Language-idiomatic alternatives:**
- **TypeScript:** Object literal implementing a factory interface
- **Python:** Module-level factory functions or a factory dict
- **Go:** Interface + struct implementing it
- **Rust:** Trait with associated types

---

## Builder

**Intent:** Construct complex objects step by step, allowing different representations from
the same construction process.

**Code smells it solves:**
- Constructor with 5+ parameters (many optional)
- Object creation requires a specific sequence of steps
- Same construction logic duplicated to create slightly different variants
- Complex configuration objects assembled inline

**When to apply during refactoring:**
- You see `new Config(true, false, null, 'abc', undefined, 3, true)` — positional args nightmare
- Object construction spans 20+ lines with conditional steps
- Test setup is painful because of complex object creation

**When NOT to apply:**
- Simple objects with 2–3 required fields — Builder is pure ceremony
- Immutable value objects with few fields — prefer a record, struct, or dataclass

**Language-idiomatic alternatives:**
- **TypeScript/JavaScript:** Options object `{ port: 8080, host: 'localhost', ... }` — often sufficient
- **Python:** `@dataclass` with defaults, or keyword arguments
- **Rust:** Builder pattern is standard (e.g., `reqwest::Client::builder()`)
- **Go:** Functional options pattern (`WithPort(8080)`)
- **Kotlin:** Named arguments + default values

---

## Prototype

**Intent:** Create new objects by copying an existing object (prototype) rather than
constructing from scratch.

**Code smells it solves:**
- Complex object initialization that's expensive to repeat
- Objects that differ from a "base" by only a few properties
- Deep cloning logic scattered or duplicated

**When to apply during refactoring:**
- Test fixtures: create a base object, clone and modify for each test case
- Configuration presets: base config cloned and tweaked per environment
- Game/simulation entities: spawn new objects from templates

**When NOT to apply:**
- Objects are cheap to create — cloning adds complexity with no payoff
- Deep-copying a complex object graph is error-prone and a rebuild is simpler

**Language-idiomatic alternatives:**
- **JavaScript:** `structuredClone()`, spread operator `{ ...obj, modified: value }`
- **Python:** `copy.deepcopy()` or `dataclasses.replace()`
- **Rust:** `Clone` trait (`#[derive(Clone)]`)
- **Java/Kotlin:** `Cloneable` interface, `.copy()` for data classes

---

## Singleton

**Intent:** Ensure a class has only one instance and provide a global point of access to it.

**Code smells it solves:**
- Global variables that manage shared state
- Multiple instances of a resource that should be shared (DB connections, loggers)
- Race conditions from multiple initialization of the same resource

**When to apply during refactoring:**
- Database connection pool, logger, or configuration manager needs exactly one instance
- You're replacing scattered global variables with controlled access

**Caution — common anti-pattern:**
Singleton is the most overused and misused pattern. Before applying, consider:
- Can you use dependency injection instead? (Almost always yes)
- Does this create hidden coupling? (Usually yes)
- Does this make testing harder? (Usually yes — hard to mock globals)

**When NOT to apply:**
- Dependency injection would work — it almost always would
- The code under it needs unit tests: singletons make tests order-dependent and hard to isolate
- Multiple instances might be needed later — this is a premature constraint
- Tell-tale signs it has already gone wrong: `getInstance()` calls scattered through business logic, tests that only pass in a particular order

**When Singleton is actually appropriate:**
- Hardware resource managers (print spooler, GPU context)
- Application-wide configuration loaded once at startup
- Connection pools where multiple instances would waste resources

**Language-idiomatic alternatives:**
- **TypeScript/JavaScript:** Module-level instance (ES modules are singletons by default)
- **Python:** Module-level instance, or `__new__` override
- **Go:** `sync.Once` + package-level variable
- **Rust:** `lazy_static!` or `once_cell::sync::Lazy`

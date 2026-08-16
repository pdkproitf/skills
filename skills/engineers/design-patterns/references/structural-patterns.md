# Structural Design Patterns Reference

These patterns explain how to assemble objects and classes into larger structures while
keeping them flexible and efficient. During refactoring, look for tight coupling, God classes,
complex subsystems, and memory-heavy object graphs.

---

## Adapter

**Intent:** Convert the interface of a class into another interface that clients expect.
Lets classes work together that couldn't otherwise because of incompatible interfaces.

**Code smells it solves:**
- Wrapper functions that translate between two APIs
- Data transformation logic duplicated at every integration point
- Third-party library calls wrapped in try/catch with format conversion

**When to apply during refactoring:**
- Integrating a new library that replaces an old one (adapt new API to old interface)
- Unifying multiple data sources with different formats
- Bridging legacy code with modern modules

**When NOT to apply:**
- The mismatch is trivial (a rename) — just rename it
- You control both sides and can align the interfaces directly

---

## Bridge

**Intent:** Separate an abstraction from its implementation so that the two can vary
independently.

**Code smells it solves:**
- Class explosion from combining two dimensions (e.g., Shape × Renderer, Message × Channel)
- Inheritance hierarchies that grow in two directions
- Platform-specific code embedded in business logic

**When to apply during refactoring:**
- You have `EmailNotificationHTML`, `EmailNotificationText`, `SmsNotificationHTML`, etc.
- You need to support multiple backends/platforms for the same abstraction
- You want to swap implementations at runtime

**When NOT to apply:**
- Only one dimension actually varies — Bridge is pure indirection
- The abstraction and implementation will not evolve independently

---

## Composite

**Intent:** Compose objects into tree structures to represent part-whole hierarchies.
Let clients treat individual objects and compositions uniformly.

**Code smells it solves:**
- Recursive data structures with `if (isLeaf) ... else (iterate children)` checks everywhere
- Separate handling logic for single items vs. collections
- Tree traversal code duplicated across multiple operations

**When to apply during refactoring:**
- File system operations (files and directories share an interface)
- UI component trees (a container is a component that holds components)
- Organization hierarchies, menu structures, expression trees

**When NOT to apply:**
- The hierarchy is flat and will stay flat — a list is simpler
- Type safety matters and uniform treatment would hide differences that callers need

---

## Decorator

**Intent:** Attach additional responsibilities to an object dynamically. A flexible
alternative to subclassing for extending functionality.

**Code smells it solves:**
- Subclass explosion: `LoggingService`, `CachingLoggingService`, `AuthLoggingService`...
- Boolean flags that enable/disable features: `doThing(data, { cache: true, log: true, retry: true })`
- Copy-pasted wrapper logic (logging, caching, timing, auth) around core functions

**When to apply during refactoring:**
- Adding cross-cutting concerns (logging, caching, retry, auth, metrics) to existing code
- Needing different combinations of behaviors without a class per combination
- Wrapping third-party objects with additional behavior

**When NOT to apply:**
- The behavior is always applied — put it in the class directly
- The stack is already deep enough to be hard to debug — a decorator chain obscures the call path

**Language-idiomatic alternatives:**
- **Python:** `@decorator` syntax is native
- **TypeScript:** Higher-order functions, or TC39 decorators
- **Go:** Middleware pattern (HTTP handler wrapping)
- **Rust:** The newtype pattern + `Deref` trait

---

## Facade

**Intent:** Provide a simplified interface to a complex subsystem.

**Code smells it solves:**
- Client code interacts with 5+ classes/modules to accomplish one task
- Initialization sequences repeated in multiple places
- Business logic tangled with subsystem orchestration

**When to apply during refactoring:**
- Simplifying a legacy API for new consumers
- Hiding complex library initialization behind a simple interface
- Creating a clean boundary between subsystems

**When NOT to apply:**
- The subsystem is already simple — the Facade is a useless layer
- Callers legitimately need the subsystem's detail, and the Facade would hide it

---

## Flyweight

**Intent:** Share common state among many objects to reduce memory usage.

**Code smells it solves:**
- Thousands of similar objects each carrying duplicate data
- Memory profiler shows high allocation for near-identical objects
- Object pools with heavy per-instance overhead

**When to apply during refactoring:**
- Text editors (character objects sharing font/style metadata)
- Game engines (thousands of particles sharing texture/mesh data)
- Document rendering (shared formatting objects)

**When NOT to apply:**
- Object count is small — this is premature optimization
- Managing the extrinsic state costs more than the memory it saves

**Language-idiomatic alternatives:**
- **JavaScript:** String interning is automatic; object references + `Map` for custom flyweights
- **Python:** `__slots__`, `sys.intern()` for strings
- **Java:** `Integer.valueOf()` caches -128 to 127 (built-in flyweight)

---

## Proxy

**Intent:** Provide a substitute or placeholder for another object to control access to it.

**Code smells it solves:**
- Expensive objects loaded eagerly when they might not be needed
- Access control checks duplicated at every call site
- Logging/monitoring code wrapped around service calls

**When to apply during refactoring:**
- **Lazy loading:** Don't create the real object until first access
- **Access control:** Check permissions before delegating
- **Caching proxy:** Return cached results for repeated calls
- **Logging proxy:** Record all calls for debugging/auditing
- **Remote proxy:** Hide network communication behind a local interface

**When NOT to apply:**
- The proxying overhead outweighs what it buys
- The language or framework already handles it more cleanly — Python decorators, Spring AOP, `functools.cached_property`

**Language-idiomatic alternatives:**
- **JavaScript:** `Proxy` object (ES6 built-in)
- **Python:** `__getattr__` / descriptors / `functools.lru_cache`
- **Go:** Embed the real struct in the proxy struct
- **Rust:** `Deref` trait for transparent proxying

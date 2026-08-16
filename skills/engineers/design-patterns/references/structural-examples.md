# Structural Design Patterns — Examples

Structure diagrams, refactoring steps, and before/after code for each pattern in
`structural-patterns.md`. Load only in `apply` mode, for the pattern actually being
applied — `design` and `review` never name classes or methods, so this content is dead
weight for them. Read only the `## <Pattern>` section you need.

---

## Adapter

**Structure:**
```
Target (interface the client expects)
└── request()

Adapter (implements Target, wraps Adaptee)
├── adaptee: Adaptee
└── request() → translates and calls adaptee.specificRequest()

Adaptee (existing class with incompatible interface)
└── specificRequest()
```

**Refactoring steps:**
1. Identify the interface your code expects (Target)
2. Identify the incompatible class or API (Adaptee)
3. Create an adapter class that implements Target and wraps Adaptee
4. Replace direct Adaptee usage with the Adapter
5. Client code now uses only the Target interface

**Example — Before:**
```typescript
// Old analytics API used everywhere
analytics.track('page_view', { page: '/home', userId: user.id });

// New analytics library has different API
newAnalytics.logEvent({
  eventName: 'page_view',
  properties: { page: '/home' },
  user: { identifier: user.id },
});
```

**Example — After:**
```typescript
class AnalyticsAdapter implements Analytics {
  constructor(private newAnalytics: NewAnalyticsLib) {}

  track(event: string, data: Record<string, any>): void {
    const { userId, ...properties } = data;
    this.newAnalytics.logEvent({
      eventName: event,
      properties,
      user: { identifier: userId },
    });
  }
}
// All existing code continues using analytics.track() unchanged
```

---

## Bridge

**Structure:**
```
Abstraction
├── implementation: Implementation   ← composition, not inheritance
└── operation()  → delegates to implementation.operationImpl()

Implementation (interface)
└── operationImpl()

ConcreteImplementationA / ConcreteImplementationB
```

**Refactoring steps:**
1. Identify the two independent dimensions of variation
2. Extract an interface for one dimension (the implementation)
3. Make the other dimension (the abstraction) hold a reference to the implementation
4. Replace inheritance with composition

---

## Composite

**Structure:**
```
Component (interface)
├── operation()
├── add(child)?
└── remove(child)?

Leaf (implements Component)
└── operation()    ← does the actual work

Composite (implements Component)
├── children: Component[]
└── operation()    ← delegates to each child
```

**Refactoring steps:**
1. Identify the leaf and composite objects
2. Extract a common interface (Component) with the shared operation
3. Implement the interface on both leaf and composite classes
4. Composite's operation iterates children and delegates
5. Replace all `instanceof` / type checks in client code with polymorphic calls

---

## Decorator

**Structure:**
```
Component (interface)
└── operation()

ConcreteComponent
└── operation()    ← core behavior

BaseDecorator (implements Component, wraps Component)
├── wrapped: Component
└── operation() → wrapped.operation()

ConcreteDecoratorA extends BaseDecorator
└── operation() → extra behavior + super.operation()
```

**Example — Before:**
```typescript
async function fetchData(url: string, useCache: boolean, enableLogging: boolean) {
  if (enableLogging) console.log(`Fetching ${url}`);
  if (useCache) {
    const cached = cache.get(url);
    if (cached) return cached;
  }
  const result = await http.get(url);
  if (useCache) cache.set(url, result);
  if (enableLogging) console.log(`Fetched ${url}: ${result.status}`);
  return result;
}
```

**Example — After:**
```typescript
// Core
const baseFetcher: Fetcher = { fetch: (url) => http.get(url) };

// Decorators (compose as needed)
const withLogging = (fetcher: Fetcher): Fetcher => ({
  fetch: async (url) => {
    console.log(`Fetching ${url}`);
    const result = await fetcher.fetch(url);
    console.log(`Fetched ${url}: ${result.status}`);
    return result;
  },
});

const withCache = (fetcher: Fetcher, cache: Cache): Fetcher => ({
  fetch: async (url) => {
    const cached = cache.get(url);
    if (cached) return cached;
    const result = await fetcher.fetch(url);
    cache.set(url, result);
    return result;
  },
});

// Compose
const fetcher = withLogging(withCache(baseFetcher, cache));
```

---

## Facade

**Structure:**
```
Facade
└── simpleOperation()  ← coordinates subsystem classes internally

SubsystemA, SubsystemB, SubsystemC
└── (complex internal APIs hidden behind the Facade)
```

**Refactoring steps:**
1. Identify the repeated sequence of subsystem calls in client code
2. Create a facade class with a single method that performs the sequence
3. Move subsystem initialization into the facade
4. Replace scattered subsystem calls with facade method calls
5. Client code now depends only on the facade, not the subsystem internals

---

## Flyweight

**Structure:**
```
Flyweight (shared, immutable state — "intrinsic state")
└── operation(extrinsicState)    ← unique state passed in, not stored

FlyweightFactory
└── getFlyweight(sharedState): Flyweight   ← returns cached instance
```

**Refactoring steps:**
1. Identify which fields are shared across many instances (intrinsic) vs. unique (extrinsic)
2. Move intrinsic state into a flyweight class (immutable)
3. Create a factory that caches and reuses flyweight instances
4. Pass extrinsic state as method parameters instead of storing it

---

## Proxy

**Structure:**
```
Subject (interface)
└── request()

RealSubject (implements Subject)
└── request()    ← the actual work

Proxy (implements Subject, holds reference to RealSubject)
└── request()    ← controls access, then delegates to realSubject.request()
```

**Refactoring steps:**
1. Extract an interface from the class you want to proxy
2. Create the proxy class implementing the same interface
3. Add the control logic (lazy init, access check, caching, logging)
4. Delegate to the real subject after the control logic
5. Replace direct references to the real subject with the proxy


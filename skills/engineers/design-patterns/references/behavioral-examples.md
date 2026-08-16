# Behavioral Design Patterns — Examples

Structure diagrams, refactoring steps, and before/after code for each pattern in
`behavioral-patterns.md`. Load only in `apply` mode, for the pattern actually being
applied — `design` and `review` never name classes or methods, so this content is dead
weight for them. Read only the `## <Pattern>` section you need.

---

## Chain of Responsibility

**Structure:**
```
Handler (interface)
├── setNext(handler): Handler
└── handle(request): Result

ConcreteHandlerA → ConcreteHandlerB → ConcreteHandlerC
Each either handles the request or passes it to next
```

**Refactoring steps:**
1. Identify the chain of checks/processing steps in the monolithic function
2. Extract each step into a handler class implementing a common interface
3. Each handler has a `next` reference and delegates if it can't handle
4. Compose the chain at configuration time
5. Client code sends the request to the first handler only

**Example — Before:**
```typescript
function processRequest(req: Request): Response {
  // Auth check
  if (!req.headers.auth) return unauthorized();
  if (!isValidToken(req.headers.auth)) return forbidden();

  // Rate limiting
  if (rateLimiter.isExceeded(req.ip)) return tooManyRequests();

  // Validation
  if (!req.body.name) return badRequest('name required');
  if (!req.body.email) return badRequest('email required');

  // Business logic
  return createUser(req.body);
}
```

**Example — After:**
```typescript
const pipeline = new AuthHandler()
  .setNext(new RateLimitHandler())
  .setNext(new ValidationHandler(['name', 'email']))
  .setNext(new CreateUserHandler());

function processRequest(req: Request): Response {
  return pipeline.handle(req);
}
// Adding a new step: just insert a handler into the chain
```

---

## Command

**Structure:**
```
Command (interface)
├── execute()
└── undo()  (optional)

ConcreteCommand
├── receiver: Receiver
├── execute()  → calls receiver methods
└── undo()     → reverses the action

Invoker
└── executeCommand(command)  ← stores history for undo

Receiver
└── (the actual business logic)
```

**Refactoring steps:**
1. Identify the action that needs to be encapsulated
2. Create a command interface with `execute()` (and optionally `undo()`)
3. Implement concrete commands that hold the receiver and parameters
4. Replace direct method calls with command creation and execution
5. Add an invoker that manages command history for undo

---

## Iterator

**Refactoring steps:**
1. Identify manual index-based loops over custom data structures
2. Implement the language's iterator protocol on the data structure
3. Replace manual loops with `for...of`, `for...in`, or equivalent
4. If multiple traversal orders are needed, create named iterator methods

---

## Mediator

**Structure:**
```
Mediator (interface)
└── notify(sender, event)

ConcreteMediator
├── componentA, componentB, componentC
└── notify(sender, event)  → coordinates the reaction

Component
├── mediator: Mediator
└── (notifies mediator instead of talking to other components)
```

**Refactoring steps:**
1. Identify the tightly coupled classes and their interactions
2. Create a mediator interface with a `notify()` method
3. Move coordination logic from the coupled classes into the mediator
4. Each class only knows about the mediator, not about other classes
5. Classes notify the mediator of events; mediator decides what happens

---

## Memento

**Structure:**
```
Originator (the object whose state you want to save)
├── save(): Memento      ← creates a snapshot
└── restore(memento)     ← restores from snapshot

Memento (opaque state container)
└── (stores internal state — not accessible to others)

Caretaker (manages history)
└── mementos: Memento[]  ← stores snapshots without peeking inside
```

**Refactoring steps:**
1. Identify the state that needs to be saved/restored
2. Create a memento class that stores a snapshot of that state
3. Add `save()` and `restore()` methods to the originator
4. Create a caretaker that manages the memento stack
5. Wire undo/redo to push/pop from the caretaker

---

## Observer

**Structure:**
```
Subject (Observable)
├── observers: Observer[]
├── subscribe(observer)
├── unsubscribe(observer)
└── notify()  ← iterates observers and calls update()

Observer (interface)
└── update(data)
```

**Example — Before:**
```typescript
class ShoppingCart {
  addItem(item: Item) {
    this.items.push(item);
    // Tightly coupled to every consumer
    uiWidget.updateCount(this.items.length);
    analyticsService.trackAdd(item);
    inventoryService.reserveItem(item);
    localStorage.setItem('cart', JSON.stringify(this.items));
  }
}
```

**Example — After:**
```typescript
class ShoppingCart extends EventEmitter {
  addItem(item: Item) {
    this.items.push(item);
    this.emit('itemAdded', { item, cart: this });
  }
}

// Subscribers register independently
cart.on('itemAdded', ({ cart }) => uiWidget.updateCount(cart.items.length));
cart.on('itemAdded', ({ item }) => analyticsService.trackAdd(item));
cart.on('itemAdded', ({ item }) => inventoryService.reserveItem(item));
cart.on('itemAdded', ({ cart }) => persist(cart));
```

---

## State

**Structure:**
```
Context
├── state: State
├── setState(state)
└── request()  → delegates to state.handle(this)

State (interface)
└── handle(context)

ConcreteStateA / ConcreteStateB
└── handle(context)  ← behavior specific to this state
```

**Example — Before:**
```typescript
class Document {
  publish() {
    if (this.state === 'draft') {
      if (this.currentUser.role === 'admin') {
        this.state = 'published';
      } else {
        this.state = 'moderation';
      }
    } else if (this.state === 'moderation') {
      if (this.currentUser.role === 'admin') {
        this.state = 'published';
      }
    } else if (this.state === 'published') {
      // already published, no-op
    }
  }
}
```

**Example — After:**
```typescript
interface DocumentState {
  publish(doc: Document, user: User): void;
}

class DraftState implements DocumentState {
  publish(doc: Document, user: User) {
    doc.setState(user.role === 'admin' ? new PublishedState() : new ModerationState());
  }
}

class ModerationState implements DocumentState {
  publish(doc: Document, user: User) {
    if (user.role === 'admin') doc.setState(new PublishedState());
  }
}

class PublishedState implements DocumentState {
  publish() { /* already published */ }
}
```

**Refactoring steps:**
1. Identify all states and the transitions between them
2. Draw a state diagram to visualize the machine
3. Create a state interface with methods for each action
4. Implement concrete state classes
5. Replace conditionals with delegation to the current state object
6. State objects handle transitions by calling `context.setState()`

---

## Strategy

**Structure:**
```
Context
├── strategy: Strategy
└── doWork()  → delegates to strategy.execute()

Strategy (interface)
└── execute(data): Result

ConcreteStrategyA / ConcreteStrategyB
└── execute(data): Result
```

**Example — Before:**
```typescript
function calculateShipping(order: Order, method: string): number {
  if (method === 'standard') {
    return order.weight * 1.5;
  } else if (method === 'express') {
    return order.weight * 3.0 + 5.0;
  } else if (method === 'overnight') {
    return order.weight * 5.0 + 15.0;
  }
  throw new Error(`Unknown shipping method: ${method}`);
}
```

**Example — After:**
```typescript
type ShippingStrategy = (order: Order) => number;

const shippingStrategies: Record<string, ShippingStrategy> = {
  standard:  (order) => order.weight * 1.5,
  express:   (order) => order.weight * 3.0 + 5.0,
  overnight: (order) => order.weight * 5.0 + 15.0,
};

function calculateShipping(order: Order, method: string): number {
  const strategy = shippingStrategies[method];
  if (!strategy) throw new Error(`Unknown shipping method: ${method}`);
  return strategy(order);
}
```

---

## Template Method

**Structure:**
```
AbstractClass
├── templateMethod()     ← final: defines the algorithm skeleton
├── step1()              ← concrete: shared across all subclasses
├── step2()              ← abstract: subclasses must implement
├── step3()              ← abstract: subclasses must implement
└── hook()               ← optional: subclasses can override

ConcreteClassA / ConcreteClassB
├── step2()  ← different implementation per subclass
└── step3()  ← different implementation per subclass
```

**Refactoring steps:**
1. Identify duplicated algorithm structures across functions/classes
2. Extract the common skeleton into a template method
3. Identify the varying steps
4. Make varying steps abstract or overridable
5. Create concrete implementations for each variant
6. Delete the duplicated original functions

---

## Visitor

**Structure:**
```
Visitor (interface)
├── visitElementA(element: ElementA)
├── visitElementB(element: ElementB)
└── visitElementC(element: ElementC)

Element (interface)
└── accept(visitor: Visitor)  ← calls visitor.visitElementX(this)

ConcreteVisitor (implements Visitor)
└── visitElementA, visitElementB, ...  ← operation logic per element type
```

**Refactoring steps:**
1. Define a visitor interface with a `visit` method per element type
2. Add an `accept(visitor)` method to each element class (this is the only change to elements)
3. Implement concrete visitors for each operation
4. Replace type-switch operations with visitor dispatch
5. New operations = new visitor class, no element changes needed


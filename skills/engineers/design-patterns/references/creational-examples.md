# Creational Design Patterns — Examples

Structure diagrams, refactoring steps, and before/after code for each pattern in
`creational-patterns.md`. Load only in `apply` mode, for the pattern actually being
applied — `design` and `review` never name classes or methods, so this content is dead
weight for them. Read only the `## <Pattern>` section you need.

---

## Factory Method

**Structure:**
```
Creator (abstract)
├── factoryMethod(): Product    ← subclasses override this
└── someOperation()             ← uses factoryMethod() internally

ConcreteCreatorA
└── factoryMethod(): ConcreteProductA

ConcreteCreatorB
└── factoryMethod(): ConcreteProductB
```

**Refactoring steps:**
1. Identify the constructor calls scattered through the code
2. Extract an interface/abstract class for the products being created
3. Create a factory method that returns the interface type
4. Move the creation logic into concrete creator classes
5. Replace all `new ConcreteProduct()` calls with `creator.factoryMethod()`
6. Run tests after each step

**Example — Before:**
```typescript
function createNotification(type: string, message: string) {
  if (type === 'email') {
    return new EmailNotification(message, smtpConfig);
  } else if (type === 'sms') {
    return new SmsNotification(message, twilioConfig);
  } else if (type === 'push') {
    return new PushNotification(message, firebaseConfig);
  }
  throw new Error(`Unknown type: ${type}`);
}
```

**Example — After:**
```typescript
interface Notification {
  send(): Promise<void>;
}

// Factory function (idiomatic TS — no class hierarchy needed)
const notificationFactories: Record<string, (msg: string) => Notification> = {
  email: (msg) => new EmailNotification(msg, smtpConfig),
  sms:   (msg) => new SmsNotification(msg, twilioConfig),
  push:  (msg) => new PushNotification(msg, firebaseConfig),
};

function createNotification(type: string, message: string): Notification {
  const factory = notificationFactories[type];
  if (!factory) throw new Error(`Unknown type: ${type}`);
  return factory(message);
}
```

**Extensibility:** To add a new notification type, add one entry to the map. No conditionals to update.

---

## Abstract Factory

**Structure:**
```
AbstractFactory
├── createProductA(): AbstractProductA
└── createProductB(): AbstractProductB

ConcreteFactory1 (e.g., LightThemeFactory)
├── createProductA(): LightButton
└── createProductB(): LightDialog

ConcreteFactory2 (e.g., DarkThemeFactory)
├── createProductA(): DarkButton
└── createProductB(): DarkDialog
```

**Refactoring steps:**
1. Identify groups of related objects that are created together
2. Extract abstract interfaces for each product type
3. Create a factory interface with a method for each product
4. Implement concrete factories for each family
5. Pass the factory to client code via dependency injection
6. Remove all direct `new` calls for family members from client code

---

## Builder

**Structure:**
```
Builder
├── setPartA(value)
├── setPartB(value)
├── setPartC(value)
└── build(): Product

Director (optional)
└── construct(builder): coordinates the build sequence
```

**Refactoring steps:**
1. Identify the complex constructor or creation sequence
2. Create a builder class with methods for each configurable part
3. Each method returns `this` (for fluent chaining)
4. Add a `build()` method that validates and returns the final object
5. Replace constructor calls with builder chains
6. Optionally create a Director for common configurations

**Example — Before:**
```typescript
const config = new ServerConfig(
  8080,          // port
  'localhost',   // host
  true,          // enableSSL
  '/certs',      // certPath
  null,          // keyPath — wait, is this right?
  30000,         // timeout
  true,          // enableLogging
  'info',        // logLevel
  5,             // maxRetries
);
```

**Example — After:**
```typescript
const config = new ServerConfigBuilder()
  .port(8080)
  .host('localhost')
  .ssl({ certPath: '/certs' })
  .timeout(30000)
  .logging({ enabled: true, level: 'info' })
  .maxRetries(5)
  .build();
```

---

## Prototype

**Structure:**
```
Prototype (interface)
└── clone(): Prototype

ConcretePrototype
└── clone(): creates a copy of itself
```

**Refactoring steps:**
1. Identify objects that are frequently copied with small variations
2. Implement a `clone()` method (or use language-native cloning)
3. Create prototype instances for common base configurations
4. Replace construction-from-scratch with clone-and-modify

---

## Singleton

**Refactoring steps (toward Singleton):**
1. Make the constructor private
2. Create a static instance field and `getInstance()` method
3. Replace all `new` calls with `getInstance()`

**Refactoring steps (away from Singleton — more common):**
1. Make the singleton accept its dependencies via constructor
2. Create the instance at application startup (composition root)
3. Pass it to consumers via dependency injection
4. Remove the static `getInstance()` method
5. Now it's just a regular class that happens to have one instance


---
name: define-test-case
description: Define acceptance test cases in DSL format at confirmed seams — happy paths, edge cases, error scenarios, and authorization, sequenced into a build order
metadata:
  phase: "implement"
  input: "one implementation phase or slice, plus the behaviors it must satisfy"
  output: "structured, sequenced DSL test cases in comment format checked against the coverage checklist"
---

# Define Test Cases

You are helping define automated acceptance test cases using a Domain Specific Language (DSL) approach.

## When to trigger

Use this skill when the user:
- asks to define or write acceptance test cases for a feature
- wants happy-path, edge-case, error, and authorization scenarios drafted before writing the code that satisfies them
- is starting TDD/BDD-style work and needs test cases specified first

It is also invoked by the `implement` skill, once per phase, before that phase's code is written.

**Scope it to one phase or slice, never a whole feature.** Every case needs a confirmed seam — a real public interface — so this skill runs at implementation time, when the seam is being built and can still be changed. Running it against a whole feature plan before any code exists produces cases anchored to imagined interfaces, which is the upfront dumping the Anti-patterns section forbids. A feature plan should carry plain-language *behaviors to cover*; this skill turns one phase's worth of those into seam-anchored cases.

## Core Principles

1. **Comment-First Approach**: Always start by writing test cases as structured comments before any implementation.

2. **DSL at Every Layer**: All test code — setup, actions, assertions — must be written as readable DSL functions. No direct framework calls in test files.

3. **Implicit Given-When-Then**: Structure tests with blank lines separating setup, action, and assertion phases. Never use the words "Given", "When", or "Then" explicitly.

4. **Clear, Concise Language**: Function names should read like natural language and clearly convey intent.

5. **Follow Existing Patterns**: Study and follow existing test patterns, DSL conventions, and naming standards in the codebase.

6. **Design for Mockability**: If a case needs to stub a boundary (external API, payment/email provider, time, randomness) and that boundary isn't independently injectable in the current code (no dependency injection, no per-endpoint SDK-style function), flag this as a design gap before writing the case — don't write a setup function that pretends the seam is mockable when it isn't.

## Test Case Structure

```javascript
// 1. Test Case Name Here
// Seam: <the public interface this case exercises, e.g. CheckoutService.checkout(), POST /api/checkout>

// setupFunction
// anotherSetupFunction
//
// actionThatTriggersLogic
//
// expectationFunction
// anotherExpectationFunction
```

### Structure Rules:
- **First line**: Test case name with number
- **Seam line**: The public interface under test — confirmed with the user before the case is written (see Workflow Step 2)
- **Setup phase**: Functions that arrange test state (no blank line between them)
- **Blank line**: Separates setup from action
- **Action phase**: Function(s) that trigger the behavior under test
- **Blank line**: Separates action from assertions
- **Assertion phase**: Functions that verify expected outcomes (no blank line between them)

## Worked Example

```javascript
// 1. User can checkout with a valid cart
// Seam: CheckoutService.checkout()

// userIsLoggedIn
// cartHasItems([{ price: 10 }, { price: 5 }])
//
// userSubmitsCheckout()
//
// expectOrderConfirmed()
// expectOrderTotalEquals(15)

// 2. Checkout fails when payment is declined
// Seam: CheckoutService.checkout()

// userIsLoggedIn
// cartHasItems([{ price: 20 }])
// paymentWillBeDeclined()
//
// userSubmitsCheckout()
//
// expectCheckoutFailed()
// expectOrderNotCreated()

// 3. Checkout is blocked for unauthenticated users
// Seam: POST /api/checkout

// userIsLoggedOut
// cartHasItems([{ price: 10 }])
//
// userSubmitsCheckout()
//
// expectResponseStatus(401)
// expectOrderNotCreated()
```

Note case 1: the expectation asserts a concrete literal (`15`), not a recomputation of the cart sum — see Anti-patterns below.

## One Case, One Behavior

A case may call **multiple assertion functions** as long as they all verify **one behavior**. Case 1 above does this correctly: `expectOrderConfirmed()` and `expectOrderTotalEquals(15)` are two facts about the same outcome — the checkout succeeded, with the right total. That's one logical assertion, expressed as two calls.

Two assertions verifying **two different behaviors** means two cases, not one:

```javascript
// BAD — one case asserting two unrelated behaviors
// 1. User can checkout with a valid cart

// userIsLoggedIn
// cartHasItems([{ price: 10 }])
//
// userSubmitsCheckout()
//
// expectOrderConfirmed()
// expectConfirmationEmailSent()   // a second behavior smuggled into case 1
```

```javascript
// GOOD — split into two cases, each independently implementable
// 1. User can checkout with a valid cart

// userIsLoggedIn
// cartHasItems([{ price: 10 }])
//
// userSubmitsCheckout()
//
// expectOrderConfirmed()

// 2. Checkout sends a confirmation email

// userIsLoggedIn
// cartHasItems([{ price: 10 }])
//
// userSubmitsCheckout()
//
// expectConfirmationEmailSent()
```

Ask: "is this a second fact about the same outcome, or a second outcome?" If the second assertion could fail for a reason unrelated to the first, split the case.

## Anti-patterns

- **Implementation-coupled assertions** — naming an assertion after an internal call or mock instead of an observable outcome (`expectPaymentServiceCalledWith`, not `expectOrderConfirmed`). The tell: the assertion would need to change on a refactor even though behavior didn't. Assert only through the seam declared for the case.
- **Tautological expectations** — the expected value restates how the code computes it (`expectOrderTotalEquals(sumOfCartItems(cart))`) instead of a concrete, independently-known literal (`expectOrderTotalEquals(15)`). A tautological expectation passes by construction and can never disagree with the code.
- **Side-channel verification** — bypassing *your own* public interface to inspect *your own* internal state, e.g. querying the database directly instead of calling `getOrder()` (`expectRow('orders', { id })` instead of `expectOrderRetrievable(id)`). The tell: the assertion reaches around the seam instead of through it.
  - **Not the same as an external-system effect.** Verifying that *another* system was affected — `expectOrderInSage`, `expectCustomerBecamePartnerInExigo` — is a legitimate observable outcome for an acceptance test; that external system's state *is* the behavior being promised. The distinction is whose internals you're peeking at: yours (forbidden) vs. a downstream system's (the point of the test).
- **Upfront dumping** — writing every conceivable case in one giant pass with no build order. Sequence cases so the first is independently implementable before the next is written; each case is a tracer bullet, not part of a bulk spec.
- **Mechanism-named cases** — a case name that describes HOW instead of WHAT (`"Checkout calls paymentService.process"` instead of `"User can checkout with a valid cart"`). The case name is a capability statement — read it without the body and it should still describe a promise the system keeps, not a step it performs internally.

## Naming Conventions

### Setup Functions (Arrange)
- Describe state being created: `userIsLoggedIn`, `cartHasThreeItems`, `databaseIsEmpty`
- Use present tense verbs: `createUser`, `seedDatabase`, `mockExternalAPI`
- **Mock only true external boundaries** — third-party APIs, payment/email providers, time, randomness. Never design a setup function that mocks an internal collaborator or module you own (e.g. `mockOrderValidator` is a red flag) — that bakes implementation-coupling into the case before any code exists.

### Action Functions (Act)
- Describe the event/action: `userClicksCheckout`, `orderIsSubmitted`, `apiReceivesRequest`
- Use active voice: `submitForm`, `sendRequest`, `processPayment`

### Assertion Functions (Assert)
- Start with `expect`: `expectOrderProcessed`, `expectUserRedirected`, `expectEmailSent`
- Be specific: `expectOrderInSage`, `expectCustomerBecamePartnerInExigo`
- Include negative cases: `expectNoEmailSent`, `expectOrderNotCreated`

## Coverage Checklist

This is a **prompt for completeness of thinking, not a quota**. You can't test everything — that's why seams are confirmed with the user first (Workflow Step 2). Within the confirmed seams, check whether each category below applies; skip a category with a one-line reason rather than inventing a case to fill it.

1. **Happy Paths** — standard successful flows
2. **Edge Cases & Boundary Conditions** — unusual but valid inputs, max/min values, empty states
3. **Error Scenarios** — invalid inputs, service failures, timeouts
4. **Permission/Authorization** — access control scenarios

## Build Order

List cases in the order they'll be implemented, not by category. Each case must be independently implementable before the next is written — one seam, one case, one minimal implementation, repeat. This is what makes each case a **tracer bullet** that informs the next, instead of a bulk spec written against imagined behavior.

Mark the next case to implement:

```
1. User can checkout with a valid cart          ← Next
2. Checkout fails when payment is declined
3. Checkout is blocked for unauthenticated users
```

Re-order or insert cases as understanding changes between cycles — the list is a working backlog, not a fixed plan committed to upfront.

## Workflow

### 1. Load Context
Load context per `# WORKSPACE` → **Context Loading** (Tier 0 + Tier 1 for this feature) if that convention is present in this project; otherwise proceed directly to Step 2. Reading `docs_context`/`system_context` first means case names and DSL vocabulary match the project's existing domain language, and any documented invariants or ADRs in this area are respected rather than contradicted by a new case.

### 2. Understand the Slice
If a spec exists, start from its **Testing Strategy** → **Behaviors to Cover**, taking only the entries this phase is responsible for — those are the agreed behaviors, but they carry no seams, so you still confirm the seams below. Add behaviors the plan missed; drop any this phase does not touch.

Ask clarifying questions about:
- What functionality is being tested
- Which systems/services are involved
- Expected behaviors and outcomes
- Edge cases and error conditions
- **Which seams (public interfaces) will be tested — confirm before writing any case; no case is written against an unconfirmed seam**

### 3. Research Existing Test Patterns
**IMPORTANT**: Before writing any test cases, search for:
- Existing acceptance/integration test files
- Current DSL function naming conventions
- Test structure patterns used in this project
- Existing DSL functions that can be reused
- How tests are organized and grouped

### 4. Define Test Cases in Comments
Write each test case in the structured comment format, checked against the Coverage Checklist, then sequenced per Build Order. Split any case that verifies more than one behavior (see One Case, One Behavior).

### 5. Identify Required DSL Functions
List all DSL functions needed:
- **Setup functions**: Functions that arrange test state
- **Action functions**: Functions that trigger the behavior under test
- **Assertion functions**: Functions that verify expected outcomes

### 6. Confirm with User
Present the sequenced test cases before any implementation and get confirmation — including the Build Order, so the user agrees on what gets implemented first.

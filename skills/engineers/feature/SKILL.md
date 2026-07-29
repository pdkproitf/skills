---
name: feature
description: Feature planning — research codebase, design options, write a structured spec/implementation plan to the project specs directory
metadata:
  phase: "plan"
  input: "[adw_id] <prompt> — adw_id is optional; prompt is a plain-language description of the feature"
  output: "path to the written spec file in specs_dir"
  dependencies: "onboard-project"
---


## When to trigger

Use this skill when the user:
- asks to plan a feature or write an implementation spec
- suggests implementing something and no spec exists yet

# Feature Planning

Generate a structured implementation plan for a feature and save it as a markdown file in the project's specs directory.

## Input Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| `adw_id` | (optional) Unique identifier for the AI Developer Workflow | `adw-42` |
| `prompt` | A plain-language description of the feature to plan | `"Add retry logic to the API client"` |

> **If `prompt` is missing or unclear, stop immediately and ask the user to provide it before doing anything else. `adw_id` is optional — if not provided, omit it from the filename and metadata.**

---

## Before You Start

### Check Argument Validity
- Confirm `prompt` describes a concrete feature (not a question or vague idea)
- If `prompt` fails this check, ask the user to clarify before continuing
- `adw_id` is optional; if not provided, proceed without it

---

## Process

Follow these steps **in order**. Do not skip ahead.

### Step 1: Load Project Context

1. If this is a new session and project context is not yet loaded, invoke the `onboard-project` skill before continuing
   - onboard-project loads project context and `# WORKSPACE` rules automatically
2. Use context loaded by onboard-project for the research phase

### Step 2: Research & Design

1. Load context per `# WORKSPACE` → **Context Loading**, matching Tier 1 on `Keywords` against this feature request — an existing doc may already cover a related or overlapping feature.
2. Create a checklist of everything that needs to be explored.
3. Run sub-tasks in parallel where possible.
4. Wait for **all** sub-tasks to finish before writing anything.
5. Collect the architectural conventions that bear on this feature — from `CLAUDE.md`, `docs_context`, matched `core_docs_dir` files, and feedback memory (e.g. thin-model, jobs-orchestrate-only). These populate the plan's **Conventions to Follow** section.
6. Present findings with 2–3 design options, each with clear pros and cons. Note if a matched dictionary entry overlaps with this feature. Get confirmation on the chosen approach before moving to Step 3.

### Step 3: Write the Plan

Use `specs_dir` (default: `docs/specs/`).

Save the file using this naming pattern:
```
{specs_dir}{unix_timestamp}-feature-{descriptive-name}.md
```
If `adw_id` was provided:
```
{specs_dir}{unix_timestamp}-feature-{adw_id}-{descriptive-name}.md
```

Replace `{descriptive-name}` with a short, hyphenated name derived from the feature (e.g., `add-retry-logic`, `create-workflow-api`, `implement-agent-logging`).

### Step 4: Enumerate Behaviors to Cover

List the behaviors this feature must be tested against — one line each, in plain language, across happy paths, edge cases, error scenarios, and permission/authorization. Skip a category with a one-line reason rather than inventing an entry to fill it.

Write these into the **Testing Strategy** → **Behaviors to Cover** section of the plan.

**Do not write DSL test cases here.** A DSL case requires a confirmed seam (the specific public interface under test), and this plan deliberately does not decide class or method placement — see Constraint 4. Seam-anchored cases are drafted by the `implement` skill, per phase, once the seam exists. What belongs in the plan is *what must hold true*, not *where it is asserted*.

---

## Plan Format

```md
# Feature: <feature name>

## Metadata
- **adw_id:** `{adw_id if provided, otherwise omit this line}`
- **prompt:** `{prompt}`
- **created:** `{timestamp}`

---

## Feature Description
<What this feature does and why it matters. 2–4 sentences.>

## User Story
As a <type of user>,
I want to <action or goal>,
So that <the benefit or outcome>.

## Problem Statement
<The specific problem or gap this feature addresses. Be concrete — avoid vague language like "improve performance".>

## Solution Statement
<The chosen approach and why it solves the problem. Reference the design option selected in Step 2.>

---

## Relevant Files

<List every file relevant to this feature. For each, explain in one sentence why it matters.>

- `path/to/file` — <reason>

### New Files to Create

- `path/to/new_file` — <purpose>

---

## Conventions to Follow

<Architectural rules this feature must respect, pulled from `CLAUDE.md`, `docs_context`, matched `core_docs_dir` files, and feedback memory. These are a checklist to validate the implementation against — NOT step prescriptions. Do not restate structure the codebase already enforces; list only rules that plausibly bear on this feature and could be gotten wrong.>
<Pattern: `for new service, logic let think whether existing conventions apply or code that could solve the problem, and if so, try to follow them, use them. else think about new conventions and code patterns that could be useful for this feature, which are reusable, and if so, propose them to the team.`>`>
- <e.g. Thin model — business logic (validation, cancellation, orchestration) belongs in a service, not a model method>
- <e.g. Jobs orchestrate only — load, guard, call service, handle result; no business logic in the job>
- <convention> — <where it applies in this feature>

---

## Implementation Plan

### Phase 1: Foundation
<Scaffolding, config changes, or prerequisite work needed before the main feature can be built.>

### Phase 2: Core Implementation
<The primary feature work — the logic, functions, classes, or APIs being added.>

### Phase 3: Integration
<Wiring the new code into the existing system — imports, registrations, config flags, etc.>

---

## Step-by-Step Tasks

> Execute every task in order, top to bottom. Do not skip steps.

### 1. <Task Name>
- <Specific, concrete action>

### 2. <Task Name>
- <Specific, concrete action>

---

## Testing Strategy

### Behaviors to Cover

<One line per behavior that must hold, in plain language — no seams, no DSL, no assertion function names. These become seam-anchored DSL cases during implementation, via the `define-test-case` skill. Note any category that does not apply, with a one-line reason.>

**Happy paths**
- <e.g. A user with a valid cart can complete checkout and gets a confirmed order>

**Edge cases & boundary conditions**
- <e.g. Checkout with an empty cart>

**Error scenarios**
- <e.g. The payment provider declines>

**Permission / authorization**
- <e.g. An unauthenticated user cannot check out>

### Unit Tests
<What units of logic need tests, and what each test should verify.>

---

## Acceptance Criteria

- [ ] <Criterion 1>
- [ ] <Criterion 2>

---

## Validation Commands

```bash
# <What this checks>
<command>
```

---

## Notes
<Optional: future considerations, known limitations, required libraries, or dependencies on other features.>
```

---

## Output

Return the full file path of the created plan, e.g.:
```
docs/specs/1711234567-feature-add-retry-logic.md
docs/specs/1711234567-feature-adw-42-add-retry-logic.md
```

---

## Constraints

1. **Resolve everything before writing** — No open questions in the final plan.
2. **Match existing conventions** — Mirror the naming, file organization, and API patterns found in the codebase. Do not prescribe function-level structure (method length, class boundaries, helper extraction) — that's decided at implementation time, not planning time.
3. **Incremental and testable** — Each phase should be shippable and verifiable on its own.
4. **Be specific — about *what* and *where*, not *how*** — Avoid vague tasks like "update the config". Name the file and the behavior that changes. But "where" means file + behavior, NOT layer placement: do not decide model-vs-service-vs-job or invent a specific class/method to add — that is a class-boundary decision (see constraint 2) and must follow the **Conventions to Follow** section, resolved at implementation time. Write "cancel sibling jobs when one fails" (behavior), not "add `PublishClipJob.cancel_siblings` class method" (structure).
5. **Fail loudly** — If a required tool is missing or an argument is invalid, stop and say so clearly.

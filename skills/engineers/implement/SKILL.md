---
name: implement
description: Implement an approved spec — reads plan from specs_dir, executes phase by phase, updates checkboxes, verifies after each phase, and commits completed work
metadata:
  phase: "implement"
  input: "path to the spec file (e.g. docs/specs/1234567-feature-name.md)"
  output: "implemented feature with updated spec checkboxes, verification results, and committed phases"
  dependencies: "onboard-project, define-test-case, commit"
---

# Implement Plan

Implement an approved spec from `specs_dir`. Execute every step in order, verify after each phase, update checkboxes as you go, and commit completed work.

## When to trigger

Use this skill when the user:
- asks to implement, build, or start coding an approved spec/plan
- says to continue, resume, or pick back up an in-progress implementation
- points to a spec file and asks what's next or to proceed with it
- asks to implement a specific phase or step from a spec

## Variables

spec_file: $ARGUMENTS — path to the spec (e.g. `docs/specs/feature-to-bootstrap-package.md`)

---

## Step 1 — Load Project Context

1. If this is a new session and project context is not yet loaded, invoke the `onboard-project` skill before continuing
   - onboard-project loads project context — including `docs/CONTEXT.md` and any matching `docs/core/*.md` — and `# WORKSPACE` rules automatically
2. Use context loaded by onboard-project for the steps below

---

## Step 2 — Read and Orient

1. Read the spec file completely
2. Read all files listed under **Relevant Files** to understand the existing codebase before touching anything
3. Load context per `# WORKSPACE` → **Context Loading**, matching Tier 1 on `Files` (the spec's **Relevant Files**) or `Keywords` (its feature description) — matched docs may carry conventions this implementation must follow
4. Identify:
   - Which phases exist and which steps are already checked off (`- [x]`)
   - The first unchecked step — that is your starting point
   - The **Validation Commands** section — you will run these after each phase

If no spec path is provided, ask for one before proceeding.

---

## Step 3 — Branch Setup

Use the `branch` convention from `# WORKSPACE` (default: `feat-{short-description}` or `feat-{adw_id}-{short-description}`).

If not already on a feature branch, create one. Skip if the branch already exists.

**Always ask before creating a branch unless the spec or user explicitly says to proceed automatically.**

---

## Step 4 — Implement Phase by Phase

For each phase in the spec's **Step by Step Tasks** section:

1. **Draft the phase's test cases** — decide the seams this phase introduces or changes (the public interfaces callers will actually use), then invoke the `define-test-case` skill for this phase only, seeded by the matching entries from the spec's **Testing Strategy** → **Behaviors to Cover**. It returns seam-anchored DSL cases in build order; implement them one at a time, in that order, rather than writing every case for the phase upfront.

   Draft per phase, never once for the whole spec — batching all phases here just relocates the upfront-dumping the skill warns against. Skip this step for a phase that introduces no testable behavior (pure config, scaffolding, a rename) and say so when you report the phase.

   If `define-test-case` flags a design gap — a boundary the case needs to stub that isn't independently injectable — resolve it before implementing, not after. That is the signal to fix the seam while it is still cheap.

2. **Implement all steps in the phase** — follow the spec's *intent* (the behavior it describes). Follow it verbatim only when the step's structure is also consistent with the codebase and the conventions (see next). Adapt when the codebase has evolved since the spec was written, or when a step's prescribed structure conflicts with the **Conventions to Follow** section, the **Code quality rules** below, `# WORKSPACE` conventions, or feedback memory — surface the conflict first (see "When the plan doesn't match reality"). A spec that names a specific class/method placement is expressing intent, not a mandate to violate the thin-model / jobs-orchestrate rules.
2.1 Before creating new files, check whether existing files could be reused or extended instead. If so, follow the existing conventions and code patterns. or think hard about whether it's worth proposing new conventions and code patterns that could be useful for this feature and reusable in the future. If so, propose them to the team.
3. **Run verification** — execute the relevant commands from the spec's **Validation Commands** section, plus the cases drafted in step 1; fix any failures before marking the phase complete
4. **Update the spec** — check off completed items (`- [ ]` → `- [x]`) and mark the phase header `✅` when fully done
5. **Pause and confirm** — report what was done and ask the user to confirm before moving to the next phase
6. **Commit the phase** — invoke the `commit` skill to generate the commit message; ask for confirmation before committing

### When the plan doesn't match reality — or its own conventions

Surface mismatches immediately — never silently deviate, and never silently comply with a step that violates a convention. Two triggers:

- **Reality drift** — the codebase has changed since the spec was written (files moved, APIs differ).
- **Convention conflict** — a step prescribes structure (a class boundary, a method on a model, logic in a job) that contradicts the spec's **Conventions to Follow** section, the **Code quality rules** below, `# WORKSPACE`, or feedback memory. Example: spec says "add `cancel_siblings` class method on the model" but the thin-model rule says business logic belongs in a service.

Report either as:

```
Issue in Phase [N] — Step [X]:
Expected: <what the spec says>
Found:    <the reality drift, or the convention it conflicts with>
Impact:   <why this matters>
Proposed: <how you suggest proceeding — e.g. implement the behavior as a service instead>
```

### Code quality rules

`# WORKSPACE` → **Code** governs whether code should exist and how much of it (YAGNI, reuse, no unrequested abstractions). The rules below govern how the code that must exist is shaped — apply both:

- **Keep functions small** — aim for 10–30 lines per method; if a method grows beyond that, it is doing too much
- **Single responsibility** — each class and method should have one clear concern; name it to reflect exactly what it does
- **Function composition** — a function may call multiple other functions as long as they all belong to the same logical concern; preferred over inlining everything into one long method
- **Thin model** — avoid adding business logic to models; create a service instead; research design patterns to apply to the logic
- **Group related steps** — steps that are tightly coupled (e.g. building objects only to immediately pass them to the next call) should be grouped into a single method rather than exposed as individual steps in the orchestrator
- **Clean orchestrators** — top-level service methods should read as a sequence of high-level calls; implementation details live in helpers, not in the orchestrator
- **Naming convention** — use clear action verbs: `validate*()`, `extract*()`, `build*()`, `verify*()`, `check*()`, `process*()`, `persist*()`

---

## Step 5 — Resuming Interrupted Work

If the spec already has checkmarks when you start:
- Trust that checked items are done — do not re-implement them
- Pick up from the first unchecked `- [ ]` item
- If something looks wrong with prior work, flag it rather than silently redoing it

---

## Step 6 — Final Verification

After all phases are complete, run the full **Validation Commands** block from the spec. Every command must pass before reporting done. Fix any failures first.

Ask whether the user wants to run the project's full test suite (not just the spec's targeted commands). If yes, run it directly and fix any regressions before reporting done.

<!-- TODO: no dedicated test-runner skill exists yet — replace the manual run above with an invocation once one is built -->
<!-- TODO: no dedicated review skill exists yet — consider invoking one here, after verification and before reporting done, once one is built -->

---

## Report

While implementing, list all steps as they complete.

When all phases are complete:
- One bullet per phase summarising what was implemented
- Files created or modified
- Verification results (commands run and their outcomes)
- Output of `git diff --stat`

# feature

> Research the codebase, design options, and write a structured implementation plan ready to execute.

---

## What it does

`feature` takes a plain-language feature description and produces a complete, executable implementation plan saved to `docs/specs/`. It doesn't just outline — it researches the actual codebase, presents design options, gets your confirmation, then writes a plan specific to how your project is built.

The plan covers:
- Feature description and user story
- Relevant files (existing and new)
- Phased implementation (Foundation → Core → Integration)
- Step-by-step tasks with specific file paths
- Testing strategy — the behaviors that must hold, in plain language (`implement` turns them into seam-anchored cases via `define-test-case`, once the seams exist)
- Acceptance criteria and validation commands

---

## When to use

- Planning a new feature before implementation
- Creating a shared artifact for async review or handoff
- Breaking down complex work before invoking `implement`

---

## Install

```bash
npx skills add pdkproitf/skills@feature
```

---

## Usage

**Claude Code:**
```
/feature add retry logic to the API client
/feature adw-42 create workflow approval API
```

**Other tools:**
```
@feature [adw_id] <plain-language feature description>
```

The `adw_id` prefix (e.g. `adw-42`) is optional. If provided, it's included in the spec filename.

---

## Output

A spec file saved to `docs/specs/` following this naming pattern:

```
docs/specs/1711234567-feature-add-retry-logic.md
docs/specs/1711234567-feature-adw-42-add-retry-logic.md
```

The skill returns the full path to the created file.

---

## Dependencies

Invokes `onboard-project` automatically if project context hasn't been loaded yet in the current session.

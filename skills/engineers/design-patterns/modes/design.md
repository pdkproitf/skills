# Mode: Design (planning time)

Assess a design option or spec before code exists.

1. Read the design document, spec, or the option under discussion.
2. Map described behavior to patterns — both explicit mentions ("plugins that can be swapped" → Strategy) and implicit ones. Assess named patterns for correct application, not just presence.
3. Run `SKILL.md` Step 3 on each candidate. Read only the surviving candidates' core `## <Pattern>` sections — never the matching `-examples.md` file. Design mode never names classes or methods (see below), so the structure diagram and before/after code there are dead weight.
4. Report per surviving candidate:

```
### Pattern: <name> — <where in the design>

**Problem:** <the structural issue in the design>
**Why this fits:** <what it solves; note the language idiom if one replaces the classic form>
**Trade-offs:** <what it costs — indirection, files, learning curve>
**Confidence:** HIGH | MEDIUM | LOW
```

5. Hand the surviving patterns back to `feature` as entries for the spec's **Conventions to Follow** — phrased as a rule to validate against ("payment providers are selected via a strategy interface, not a conditional"), not as a step.

**Do not name classes or methods here.** The spec deliberately leaves class placement undecided (`feature` Constraint 4). A design-time pattern recommendation states the shape the solution should take, and is confirmed at implementation time when the seam is real.

Close with the shared **Report** format in `SKILL.md`.

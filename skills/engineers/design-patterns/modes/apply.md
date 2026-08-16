# Mode: Apply (implementation time)

Apply one pattern to code being written or restructured.

1. **Check prerequisites** — the behavior being restructured must be covered by tests. If it isn't, say so and get agreement before proceeding; on a pure refactor, missing coverage is a stop condition, not a footnote.
2. **Run `SKILL.md` Step 3** even when the user named the pattern. If it fails the fit test, say so before applying it.
3. **Load the example tier.** Read the matching `-examples.md`'s `## <Pattern>` section —
   `references/creational-examples.md`, `structural-examples.md`, or `behavioral-examples.md`.
   Modern and architectural patterns have no example file; their core section is enough. This is
   the one mode that needs the structure diagram, refactoring steps, and before/after code —
   design and review never load it.
4. **Map roles to code** — write down which existing unit fills each role in the pattern before editing anything. For Strategy: the class holding the conditional is the Context, each branch is a Concrete Strategy, the extracted abstraction is the Strategy interface.
5. **Transform incrementally** — one role at a time, each step independently testable:
   1. Extract the abstraction (interface, protocol, trait, function type)
   2. Implement one concrete variant
   3. Run tests
   4. Migrate the remaining variants one at a time
   5. Remove the original conditional
   6. Clean up
6. **Verify** — run the phase's tests and the spec's **Validation Commands**. Behavior must be unchanged unless the change was requested; public API must be unchanged unless intentional.
7. **Document the extension point** — one line stating how to add the next variant ("to add a provider, implement `PaymentProvider` and register it in the provider map"). This is what makes the pattern pay off later; without it, the indirection is just cost.

Never change more than one pattern role in a single step, and never apply two patterns in one pass.

Close with the shared **Report** format in `SKILL.md`.

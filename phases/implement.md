# Phase 4: Code Implementation

## Purpose

Write code following the implementation plan. Code is a derived artifact
from the specification — never write code without an up-to-date plan.

## Rules

1. **Follow the plan phases in order.** Respect dependencies between phases.
2. **Follow SOLID and pluggability principles** as defined in
   [solid-architecture reference](../references/solid-architecture.md).
   If `.dev_flow/rules/` defines alternative architectural conventions — project rules
   take precedence.
3. **Reference traceable IDs in code comments:**
   ```python
   # [C_ACS_03_01] PermissionInterceptor — enforcement pattern
   class PermissionInterceptor:
       ...
   ```
4. **Implement ALL fields from specification tables.** Do not skip optional fields —
   implement them with their default values.
5. **Use immutable structures** where the spec says "immutable after creation".
6. **Implement all error cases** from the spec's Errors tables.
7. **Update the plan status** after completing each phase:
   - Change `[TODO]` to `[IN PROGRESS]` when starting
   - Change `[IN PROGRESS]` to `[DONE]` when tests pass
   - Update the checkbox in the Progress section

## Gate Check Before Starting

Before writing code, verify:
- [ ] The plan covers ALL specification sections
- [ ] Technology decisions are documented with rationale
- [ ] Phase dependencies are explicitly stated
- [ ] The specification passes its self-validation checklist

**Skill check (gate).** Identify the technologies/patterns this task uses. MUST read
`.dev_flow/skills/_index.yaml` and load matching skills BEFORE writing code or doing
research. If research is still needed, save consolidated results to `.dev_flow/skills/`
after the task. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml`
and load rules for the area being modified. New code MUST comply (`must` = blocks). See
[rule phase](rule.md) and [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding).

**Role check:** If you'll delegate any step (run tests, verify, a wide search) to a
subagent, check `.dev_flow/roles/_index.yaml` first and reuse a fitting base role or
project overlay instead of re-deriving one. See [Roles](../references/roles.md).

## Implementation Workflow

```
1. Read the plan phase you're implementing
2. Read the referenced specification sections
3. Write code that satisfies ALL spec contracts
4. Add traceable ID comments linking back to concept/spec
5. Add/update tests covering spec's error cases and invariants — see [Test phase](testing.md)
6. Run functional tests (unit → mock) covering the changed code
7. Fix any test failures (code must satisfy the spec)
8. Run pre-commit review (clean-context subagent) — see [Review phase](review.md)
9. Fix any blocking issues from the review, re-run tests if needed
10. Run verification (regression, integration, live) — see [Verify phase](verify.md)
11. If Verify fails → fix code → re-run steps 6-10
12. Update plan phase status to [DONE]
13. Update plan Progress checkboxes
14. Ask user for commit approval before committing
```

## Delegation for focus

Steps 6, 8, and 10 (run tests, review, verify) flood the context with output that
*writing the code* never needs. Keep the implementation here — it needs the full plan and
spec — but delegate those verification steps to a subagent and take back only the verdict.
See **[Delegation for Focus](../references/delegation.md)**.

## Anti-Patterns

- Writing code before the spec is complete
- Implementing only the "happy path" and ignoring spec error cases
- Skipping traceable ID comments
- Not updating plan status after completion
- Adding features not described in the specification

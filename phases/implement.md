# Phase 4: Code Implementation

## Purpose

Write code following the implementation plan. Code is a derived artifact
from the specification — never write code without an up-to-date plan.

## Rules

1. **Follow the plan phases in order.** Respect dependencies between phases.
2. **Reference traceable IDs in code comments:**
   ```python
   # [C_ACS_03_01] PermissionInterceptor — enforcement pattern
   class PermissionInterceptor:
       ...
   ```
3. **Implement ALL fields from specification tables.** Do not skip optional fields —
   implement them with their default values.
4. **Use immutable structures** where the spec says "immutable after creation".
5. **Implement all error cases** from the spec's Errors tables.
6. **Update the plan status** after completing each phase:
   - Change `[TODO]` to `[IN PROGRESS]` when starting
   - Change `[IN PROGRESS]` to `[DONE]` when tests pass
   - Update the checkbox in the Progress section

## Gate Check Before Starting

Before writing code, verify:
- [ ] The plan covers ALL specification sections
- [ ] Technology decisions are documented with rationale
- [ ] Phase dependencies are explicitly stated
- [ ] The specification passes its self-validation checklist

**Skill check:** Before touching code, identify what technologies or patterns this task
uses (SDK, API, architectural pattern). Read `.dev_flow/skills/_index.yaml` and check for
relevant skills. If found — load them. If external research will be needed — note it;
save consolidated results to `.dev_flow/skills/` after the task completes. See [skill phase](skill.md).

## Implementation Workflow

```
1. Read the plan phase you're implementing
2. Read the referenced specification sections
3. Write code that satisfies ALL spec contracts
4. Add traceable ID comments linking back to concept/spec
5. Write tests covering spec's error cases and invariants
6. Run tests (if test suite exists) — see [Test phase](testing.md)
7. Update plan phase status to [DONE]
8. Update plan Progress checkboxes
9. Ask user for commit approval before committing
```

## Anti-Patterns

- Writing code before the spec is complete
- Implementing only the "happy path" and ignoring spec error cases
- Skipping traceable ID comments
- Not updating plan status after completion
- Adding features not described in the specification

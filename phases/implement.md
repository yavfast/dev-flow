# Phase 4: Code Implementation

## Purpose

Write code following the implementation plan. Code is a derived artifact from the specification — never write code without an up-to-date plan.

## Rules

1. **Follow the plan phases in order.** Respect dependencies between phases.
2. **Follow SOLID and pluggability principles** as defined in [solid-architecture reference](../references/solid-architecture.md). If `.dev_flow/rules/` defines alternative architectural conventions — project rules take precedence.
3. **Reference traceable IDs in code comments:**
   ```python
   # [C_ACS_03_01] PermissionInterceptor — enforcement pattern
   class PermissionInterceptor:
       ...
   ```
4. **Implement ALL fields from specification tables.** Do not skip optional fields — implement them with their default values.
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

**Skill check (gate).** Identify the technologies/patterns this task uses. MUST read `.dev_flow/skills/_index.yaml` and load matching skills BEFORE writing code or doing research. If research is still needed, save consolidated results to `.dev_flow/skills/` after the task. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load rules for the area being modified. New code MUST comply (`must` = blocks). See [rule phase](rule.md) and [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding).

**Knowledge activation (per-burst).** The skill/rule gate above is **re-triggered at the moment of action**, not only here at phase start: before each action burst, re-surface the applicable rules/skills beside the work with a pointer-only Pre-Action Marker — a long session drifts from what was loaded. For a high-stakes burst, escalate to the matching tier (deterministic tripwire / interface-gate / sampled cross-model verifier) where the runtime can observe the action; self-attestation is never the control. See [Application Enforcement](../references/application-enforcement.md).

**Resource check.** Before fetching a design export or external document (e.g. via the Figma MCP) for this implementation, check `.dev_flow/cache/_index.yaml` and reuse a cached copy; save an expensive new fetch back to the cache. See [Resource Cache](../references/cache.md).

**Role check:** If you'll delegate any step (run tests, verify, a wide search) to a subagent, check `.dev_flow/roles/_index.yaml` first and reuse a fitting base role or project overlay instead of re-deriving one. See [Roles](../references/roles.md).

**Framework check.** When `docs/_framework.md` exists, load it alongside `docs/_index.md` — it is the project's architectural map (core abstractions · layers · extension points · shared utilities · conventions, maintained by onboard and the [`audit code` scope](audit.md#step-9--code-scope-the-whole-codebase-audit)). New code MUST respect the layer boundaries and extension points it records; the map links down to the `.dev_flow/rules/` that carry the enforceable detail. No-op while the file is absent.

**Forecast check (advisory).** Before a material implementation decision, forecast at *implement altitude* — the blast-radius of this change and the results of the next step or two (incl. tool calls) — and route each anticipation through the YAGNI-gate. The gate is **strict** here: speculative future-proofing in code with no near-term trigger defaults to `drop + record`, not `build now`. Keep the free one-step check on every non-trivial action — *does the inevitable next step undo or absorb this one?* See [Consequence Forecasting](../references/consequence-forecasting.md).

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
14. Reflection checkpoint — harvest any rule/skill proposals (see [Reflection](#reflection--harvest-rules-and-skills) below)
15. Ask user for commit approval before committing
```

## Delegation for focus

Steps 6, 8, and 10 (run tests, review, verify) flood the context with output that *writing the code* never needs. Keep the implementation here — it needs the full plan and spec — but delegate those verification steps to a subagent and take back only the verdict. See **[Delegation for Focus](../references/delegation.md)**.

## Reflection — harvest rules and skills

At the phase/subtask boundary, before asking for commit approval, run the [Transition Checkpoint](../references/experience-capture.md) over the work item just closed. This is where the project's knowledge base improves automatically — implementation is the primary **rule auto-discovery** touchpoint:

- **Rule from a recurring pattern.** If the plan phase or the code surfaced an architectural constraint, naming convention, or error-handling pattern not yet in `.dev_flow/rules/` — write it as a rule automatically (default severity `should`, or `prefer`). See [Project Rules → Auto-discovery](../SKILL.md#project-rules) and the [rule phase](rule.md).
- **Skill from non-trivial knowledge.** If research during the task produced broadly-useful, non-obvious technology knowledge — write/update a skill through the [skill phase](skill.md) non-triviality filter, so the next task starts ahead.

**Auto-apply, no permission prompt.** A harvested lesson is written to the catalogue automatically through the rule/skill gate (the gate is structural, not a self-score). **Never auto-write a `must`**, and route any rule that would contradict an existing one to an [independent clean-context review](../references/delegation.md) — write only if confirmed. Every write is visible to the developer in the commit diff. Then distill the segment into a pinned summary, demote its raw turns, and promote durable working-memory parts to the task file. See **[Experience Capture](../references/experience-capture.md)**.

## When the Spec Pushes Back

"Code must satisfy the spec" assumes the spec is right. If implementation surfaces evidence it is not — two defensible readings of a contract, a constraint that cannot hold, a plan technology decision that does not work as assumed — do **not** pick a reading silently and do not bend the code to a document you have evidence is wrong. Stop the affected work item and escalate to the owning document: see **[Upstream Escalation](../references/escalation.md)**.

## Anti-Patterns

- Writing code before the spec is complete
- Implementing only the "happy path" and ignoring spec error cases
- Silently choosing one of two defensible spec readings instead of escalating (see [Upstream Escalation](../references/escalation.md))
- Skipping traceable ID comments
- Not updating plan status after completion
- Adding features not described in the specification
- Closing a work item without reflecting — a recurring pattern left uncaptured as an auto-applied rule/skill (see [Reflection](#reflection--harvest-rules-and-skills))

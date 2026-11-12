# Phase 4: Code Implementation

## Contents

- [Purpose](#purpose) — code as an artifact derived from the specification and the plan
- [Role Responsible](#role-responsible) — the Implementer role; the brief lines of a delegated code edit
- [Rules](#rules) — what holds for every code edit of the phase
- [Gate Check Before Starting](#gate-check-before-starting) — the Plan → Code gate, knowledge checks, the design sign-off
- [Implementation Workflow](#implementation-workflow) — the steps of one plan phase, from the plan's `Verify:` field to the commit sign-off
- [Delegation for Focus](#delegation-for-focus) — which steps go to a subagent and with which brief
- [Reflection — harvest rules, skills, and tools](#reflection--harvest-rules-skills-and-tools) — the reflection checkpoint and its filters
- [When the Spec Pushes Back](#when-the-spec-pushes-back) — upstream escalation instead of bending the code
- [Anti-Patterns](#anti-patterns) — what the phase never does

## Purpose

Write code following the implementation plan. Code is a derived artifact from the specification — never write code without an up-to-date plan.

## Role Responsible

This phase is handled by **Implementer**: [roles/implementer.ai.md](../roles/implementer.ai.md). A delegated code edit: the brief carries `Skill root: <absolute path of the skill>` and `Read first: roles/implementer.ai.md` ([SKILL.md → Subagents](../SKILL.md#subagents)).

## Rules

- **Follow the plan phases in order.** Respect dependencies between phases.
- **Follow SOLID and pluggability principles** as defined in the [solid-architecture reference](../references/solid-architecture.md). If `.dev_flow/rules/` defines alternative architectural conventions — project rules take precedence.
- **Implement ALL fields from specification tables.** Do not skip optional fields — implement them with their default values.
- **Use immutable structures** where the spec says "immutable after creation".
- **Implement all error cases** from the spec's Errors tables.
- **Promoted prototype:** copy from the prototype place only the files the plan marks `keep`; nothing else enters the task's working tree from it ([research](research.md#prototype-in-the-project)).
- **Reference traceable IDs in code comments.**

```python
# [C_ACS_03_01] PermissionInterceptor — enforcement pattern
class PermissionInterceptor:
    ...
```

- **Update the plan status** after completing each phase: `[TODO]` → `[IN PROGRESS]` when starting, `[IN PROGRESS]` → `[DONE]` when the phase's verification passes (workflow steps 6-11), and tick the checkbox in the Progress section.

## Gate Check Before Starting

**Design sign-off (stop).** Before the first code edit, the task file must show the developer's go at the design sign-off; missing → present it (DEC batch, changed docs, files to touch, proportionality block) and wait. No stop for a Trivial change (no design documents); otherwise skipped only under `Autonomy: full`. A `proposed` Design Decision counts as settled for the checklist below, but no code edit runs on one. See [SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints).

Before writing code, verify:
- [ ] The plan covers ALL specification sections
- [ ] Technology decisions are documented with rationale
- [ ] Contested technology forks are resolved (consensus + rationale) or recorded as open decisions with a resolution trigger (see [Interview Mode](../references/interview-mode.md))
- [ ] Phase dependencies are explicitly stated
- [ ] The phase you're implementing declares a `Verify:` field (what to check on completion)

**Skill check (gate).** Identify the technologies/patterns this task uses. MUST read `.dev_flow/skills/_index.yaml` and load matching skills BEFORE writing code or doing research. If research is still needed, save consolidated results to `.dev_flow/skills/` after the task. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load rules for the area being modified. New code MUST comply (`must` = blocks). See [rule phase](rule.md) and [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding).

**Intent check.** Re-read the task file's `## Intent` (goal / target state / expected result) before coding, and hold each material implementation decision against it: *does this option serve the recorded goal?* If executing the plan as written would not achieve the recorded goal — or the letter and the goal diverge — stop and surface the conflict (marked options + recommendation) instead of silently following either. See [Task Intent](../references/task-intent.md).

**Ticket check (if tied to a tracker).** If the task's Current Work Item carries a `Ticket:` link, propose the *In Progress* status transition through the discovered tracker skill/MCP before coding — confirmed first, never silent. See [Ticket Tracker Integration](../references/ticket-tracker.md).

**Verification economy.** A comparison outside this task and unasked for (code comments against docs, doc freshness, a corpus sweep) runs only on a signal; with every signal **false**, skip it, record it as `unobserved` with the absent fact, and file a `todo` if a substantive suspicion remains. A signal you cannot compute is not a false one — run the check in full. Re-reading a file already in context needs an invalidator. See [Verification Economy](../references/verification-economy.md).

**Knowledge activation (per-burst).** The skill/rule gate above is **re-triggered at the moment of action**, not only here at phase start: before each action burst, re-surface the applicable rules/skills beside the work with a pointer-only Pre-Action Marker — a long session drifts from what was loaded. For a high-stakes burst, escalate to the matching tier (deterministic tripwire / interface-gate / sampled cross-model verifier) where the runtime can observe the action; self-attestation is never the control. See [Application Enforcement](../references/application-enforcement.md).

**Resource check.** Before fetching a design export or external document (e.g. via the Figma MCP) for this implementation, check `.dev_flow/cache/_index.yaml` and reuse a cached copy; save an expensive new fetch back to the cache. See [Resource Cache](../references/cache.md).

**Design-source check (UI work).** When implementing UI from a design source of truth (design-tool link, tokens, mockups), implement **every variant and state the design defines** — not just the ones visible in the handed frame — and use the design's tokens instead of hardcoded near-matches: Verify will validate the result property by property against that source. See [Design Compliance](../references/design-compliance.md).

**Role check.** If you'll delegate any step (run tests, verify, a wide search) to a subagent, check `.dev_flow/roles/_index.yaml` first and reuse a fitting base role or project overlay instead of re-deriving one. See [Roles](../references/roles.md).

**Framework check.** When `docs/_framework.md` exists, load it alongside `docs/_index.md` — it is the project's architectural map (core abstractions · layers · extension points · shared utilities · conventions, maintained by onboard and the [`audit code` scope](audit.md#step-9--code-scope-the-whole-codebase-audit)). New code MUST respect the layer boundaries and extension points it records; the map links down to the `.dev_flow/rules/` that carry the enforceable detail. No-op while the file is absent.

**Reach check.** Before the first edit, list the places the change must reach: grep every caller of each symbol whose signature, return value, or behaviour changes, then add to the list the tests, fixtures, config, and exports bound to it. See [Impact Walk → Symbol reach](../references/impact.md#symbol-reach).

**Reuse check (advisory).** Before writing a new function, class, util, or shared constant, search for an existing one first — the `docs/_framework.md` shared-utilities map, `.dev_flow/skills/`, a symbol/behaviour search of the codebase (delegate a wide search, keep the conclusion), the standard library and platform, then an already-installed dependency; no new dependency for a few lines. Call an exact match; extend a near-match rather than fork it (unless the two cases change for different reasons); escalate if reuse needs a contract change. When writing new code, leave a reuse seam only for a real near-term consumer — speculative generality defaults to `drop + record`. The code-altitude counterpart of the concept [Reuse Check](concept.md#reuse-check). See [Code Reuse](../references/code-reuse.md).

**Forecast check (advisory).** Before a material implementation decision, forecast at *implement altitude* — the blast-radius of this change and the results of the next step or two (incl. tool calls) — and route each anticipation through the YAGNI-gate. The gate is **strict** here: speculative future-proofing in code with no near-term trigger defaults to `drop + record`, not `build now` — never a never-cut requirement ([Consequence Forecasting](../references/consequence-forecasting.md#the-yagni-gate)). Keep the free one-step check on every non-trivial action — *does the inevitable next step undo or absorb this one?* See [Consequence Forecasting](../references/consequence-forecasting.md).

**Spotted-defect reflex.** Implementing routinely surfaces a *separate, unrelated* defect in neighbouring code — distinct from speculative future-proofing of *this* change (→ `drop+record`) and from a defective spec (→ [Upstream Escalation](../references/escalation.md)). If it is out-of-scope and deferrable, don't fold it in and don't lose it: file an agent-initiated [`todo`](todo.md) (cheap subagent, or inline if trivial). If it is urgent, surface it now instead.

## Implementation Workflow

1. Read the plan phase you are implementing, including its `Verify:` field — the definition-of-done that Test and Verify check on completion.
2. Read the referenced specification sections.
3. Write code that satisfies ALL spec contracts and the phase's `Verify:` checklist.
4. Add traceable ID comments linking back to concept/spec.
5. Check the spec's error cases and invariants; choose the form of each check by [Test Economy](../references/test-economy.md#choosing-the-form) — a permanent test only with a keep reason, trivial behaviour as a temporary test; see [Test phase](testing.md).
6. After each edit, run the task-set tests (unit → mock) that cover the edited code — never the full suite ([When to run](../references/test-economy.md#when-to-run)).
7. Fix any test failures — code must satisfy the spec.
8. Run pre-commit review (clean-context subagent) — see [Review phase](review.md).
9. Fix any blocking issues from the review; re-run the task-set tests that cover the fix ([When to run](../references/test-economy.md#when-to-run)).
10. Run verification (regression, integration, live) — see [Verify phase](verify.md).
11. If Verify fails → fix code → re-run steps 6-10.
12. Update the plan phase status to [DONE].
13. Update the plan Progress checkboxes.
14. Reflection checkpoint — harvest rules/skills/tools, auto-applied through the structural gate; see [Reflection](#reflection--harvest-rules-skills-and-tools) below.
15. Commit sign-off — changed files, review verdict, intent verdict, proportionality block, unpushed commits; "Ready to commit?" and offer the push as its own question; wait ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)). On the yes at the task's last commit: [close the task](status.md#on-task-completion) before `git commit`. The commit message carries one `Keep-Test` trailer per new permanent test ([Test Economy](../references/test-economy.md#keep-reasons)).

## Delegation for Focus

Delegate steps 6, 8 and 10 (run tests, review, verify) to a subagent and take back only the verdict; keep the implementation itself in this context. Each brief carries `Skill root: <absolute path of the skill>` and its role line: `Read first: roles/tester.ai.md` for steps 6 and 10, the reviewer line of the [Review phase](review.md#pre-commit-review-procedure) for step 8. See **[Delegation for Focus](../references/delegation.md)**.

## Reflection — harvest rules, skills, and tools

At the phase/subtask boundary, before asking for commit approval, run the [Transition Checkpoint](../references/experience-capture.md) over the work item just closed.

- **Rule from a recurring pattern.** If the plan phase or the code surfaced an architectural constraint, naming convention, or error-handling pattern not yet in `.dev_flow/rules/` — write it as a rule automatically (default severity `should`, or `prefer`) — never a fact about the developer or the machine, which belongs to `.dev_flow/profile/` under its consent regime ([Adaptation Profile](../references/adaptation-profile.md)). See [Project Rules → Auto-discovery](../references/conventions.md#project-rules) and the [rule phase](rule.md).
- **Skill from non-trivial knowledge.** If research during the task produced broadly-useful, non-obvious technology knowledge — write/update a skill through the [skill phase](skill.md) non-triviality filter, so the next task starts ahead — never a fact about the developer or the machine, which belongs to `.dev_flow/profile/` under its consent regime ([Adaptation Profile](../references/adaptation-profile.md)).
- **Tool to keep.** A script written in the session workspace that you judge worth reusing — and, as a must-consider, a deterministic check or operation hand-rolled in this and at least one earlier task, or hand-rolled wrong — is [promoted](../references/project-tools.md#promote) to a project tool: header contract, real acceptance samples, your own run of every sample; it stays `unverified` until the audit tools step verifies it. A covering tool you did not use is recorded as a bypass with its reason ([Experience Capture](../references/experience-capture.md#applying-experience--auto-apply-through-structural-gates)).
- **Observation for a covered lesson.** A lesson an existing rule, skill, or tool already covers is not dropped: when its problem class recurred or the artifact was breached, append a `recurred` observation to that artifact's ledger entry (a hand-rolled check a covering tool covers: a [bypass](../references/project-tools.md#use-accounting) instead, unless one is already recorded for this check — by a failed run or an earlier rung — never a second); when the artifact fired on its sanctioned path and left a non-model by-product, append `exercised` ([Experience Capture](../references/experience-capture.md#applying-experience--auto-apply-through-structural-gates)).

**Auto-apply, no permission prompt.** A harvested lesson is written to the catalogue automatically through the rule/skill/tool gate (the gate is structural, not a self-score). **Never auto-write a `must`**, and route any rule that would contradict an existing one to an [independent clean-context review](../references/delegation.md) — write only if confirmed. Every write is visible to the developer in the commit diff. Then distill the segment into a pinned summary, demote its raw turns, and promote durable working-memory parts to the task file. See **[Experience Capture](../references/experience-capture.md)**.

## When the Spec Pushes Back

If implementation surfaces evidence the spec is wrong — two defensible readings of a contract, a constraint that cannot hold, a plan technology decision that does not work as assumed — do **not** pick a reading silently and do not bend the code to a document you have evidence is wrong. Stop the affected work item and escalate to the owning document: see **[Upstream Escalation](../references/escalation.md)**.

## Anti-Patterns

- Writing code before the spec is complete
- Implementing only the "happy path" and ignoring spec error cases
- Silently choosing one of two defensible spec readings instead of escalating (see [Upstream Escalation](../references/escalation.md))
- Skipping traceable ID comments
- Not updating plan status after completion
- Adding features not described in the specification
- Closing a work item without reflecting — a recurring pattern left uncaptured as an auto-applied rule/skill/tool (see [Reflection](#reflection--harvest-rules-skills-and-tools))

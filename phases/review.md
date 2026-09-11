# Phase 6: Review & Validation

## Purpose

Validate pipeline gates, detect conflicts between documents, manage deprecation, ensure the documentation-code alignment is maintained, and perform pre-commit code review of changes.

## Pre-Commit Code Review

**Before any commit**, a code review of the changes MUST be performed by a **subagent with a clean context** — it sees only the diff and the spec, without the implementer's accumulated assumptions.

This clean-context review also **realizes the `sampled-verifier` tier** of [Application Enforcement](../references/application-enforcement.md): for a high-stakes procedure, the fresh, cross-model reviewer judging conformance to the documented procedure (as a rubric) is exactly the external verifier that the acting agent's self-attestation can never be.

A checkable guard on the reviewer itself: **2+ consecutive substantive diffs** (non-trivial change class) reviewed with **zero actionable findings** (blocking/should-fix, not nits) is a `verifier-rubber-stamp` tripwire. Re-running the same reviewer cannot fix itself — surface to the user; switch model if the harness has one.

Its symmetric twin guards the opposite failure — a loop that never converges. Findings carry a computed **materiality**, a recurring non-`must` finding trips `review-non-convergence` and leaves the loop as a `contested` todo, and a repeat round reads only the delta from its baseline. Procedure: **[Review Convergence](../references/review-convergence.md)**.

### Pre-Commit Review Procedure

0. **Scope the round** ([Review Convergence](../references/review-convergence.md)). Round 1 reads the whole diff. A later round reads **delta ∪ carry-over ∪ always-in-scope** from the baseline written at the close of the previous round, unless a full reason holds: no reachable baseline · upstream spec/plan/rules changed · change class `architectural`. Areas declared `Criticality: critical` are always in scope. Name `scope_mode` — and the reason for a full round — in the report.

0b. **Select the round mode** (repeat rounds only — [Verification Economy](../references/verification-economy.md)). A contained fix of a prescribed finding that is neither `must` nor a security class runs as `confirm`: check the diff against the prescribed fix and that every test that was green is still green, then skip to step 5. Everything else, and round 1 always, runs `full` — continue to step 1. A `confirm` round that does not confirm replays as `full` from step 1.

1. **Launch a reviewer subagent** (role: [reviewer.ai.md](../roles/reviewer.ai.md)) with a clean context containing only:
   - The git diff of all staged/unstaged changes
   - The relevant specification (`*.sp.md`) for contract verification
   - The relevant plan (`*.plan.md`) for completeness verification
   - Project rules (`.dev_flow/rules/`) if they exist — binding
   - Relevant project skills (`.dev_flow/skills/`) for the changed area, if they exist
   - SOLID architecture reference (`references/solid-architecture.md`)

   **Pass the artifact + contract only — never the author's conclusion or self-assessment; frame the review adversarially** ("find what violates the spec/rules", not "is this good?"). Realizes the `sampled-verifier` contract ([Application Enforcement](../references/application-enforcement.md)).

2. **The reviewer subagent checks:**

   | Check | Description | Finding severity |
   |-------|-------------|------------------|
   | Spec compliance | Code implements all spec contracts, error cases, invariants | `must` |
   | Plan completeness | All plan tasks for the current phase are addressed | `must` |
   | Rules compliance | New code follows `.dev_flow/rules/` | the rule's own severity |
   | Skill pitfalls | Change doesn't reintroduce a pitfall documented in a loaded skill | `should` (`must` if also a `must` rule) |
   | SOLID compliance | Code structure follows SOLID and pluggability principles ([reference](../references/solid-architecture.md)) unless overridden by project rules | `should` |
   | No regressions | Changes don't break existing functionality | `must` |
   | No leftover artifacts | No debug code, TODOs, commented-out blocks | `should` |
   | Code quality | Naming, structure, readability follow project conventions | `prefer` |
   | Security | No obvious vulnerabilities (injection, exposure, etc.) | `must` |

   The column is the finding's **severity**, on the same `must` / `should` / `prefer` axis as project rules. It is the reviewer's input to the materiality computation in step 3 — not the verdict. A check's severity is a ceiling per finding, not a floor: a reviewer states a lower severity when the concrete finding warrants it.

3. **Classify each finding** ([Review Convergence](../references/review-convergence.md)). The reviewer assigns `severity` (`must` / `should` / `prefer`); **materiality is computed, never assigned** — from `severity`, finding type, and the effective `Criticality` declared by the owning concept/spec. `blocking` spawns the next round; `advisory` is reported only; `deferrable` leaves the loop at once. A `must` or security finding is always `blocking` and never leaves the loop.

4. **Review result:**
   - **Pass** — proceed to commit approval.
   - **Fail (blocking)** — fix issues, re-run tests if needed, then re-review. Select the repeat round's mode first: a contained fix of a prescribed finding that is neither `must` nor a security class is a `confirm` round in the main context, everything else spawns the clean-context reviewer. A `confirm` round that does not confirm replays as `full` — it never closes a finding it did not confirm. See [Verification Economy](../references/verification-economy.md). A non-`must` blocking finding that recurs unresolved into a second round trips `review-non-convergence` and routes to a `contested` todo instead of a third round.
   - **Warnings only** — present warnings to the user, proceed if user approves.

5. **Report format:**

   ```
   ## Pre-Commit Review — round {N}, scope {full|incremental}, mode {full|confirm}{, reason: {full reason}}

   **Result:** PASS / FAIL / WARNINGS

   ### Blocking Issues
   - [ ] {issue description} — {file:line} — severity {must|should}, criticality {value|unstated}, recurrence {n}

   ### Warnings
   - {warning description} — {file:line} — advisory under criticality {value|unstated}

   ### Routed to todo (contested)
   - {TD_id} — {issue} — {why it did not converge; both positions recorded in the register}

   ### Unobserved
   - {check the reviewer could not evaluate} — {what was missing: no spec for the area, no test result, no runtime access}
   - {check the reviewer did not open} — {the absent entry condition: no signal is true for this file}

   ### Summary
   {1-2 sentence summary of the review}
   ```

   A check the reviewer could not evaluate, or did not open for a missing entry condition ([Verification Economy](../references/verification-economy.md)), is listed as `unobserved` with the absent fact — it is neither a pass nor a warning, and never omitted. Every routed record is named; a silently deferred finding is a report-ceiling violation. The summary claims no more than the evidence supports. See [Evidence Discipline](../references/evidence-discipline.md).

6. **Close the round.** Write `baseline` (tree state), the round mode, `carry_over` (open blocking findings + recurrence), and `always_in_scope` into the task file, so the next round scopes from them.

## After Review: Verify Phase

After the review passes, proceed to the [Verify phase](verify.md) for regression, integration, and live testing. Do NOT commit directly after review — verification must pass first.

### Review -> Verify Gate

- [ ] Pre-commit review by a clean-context subagent passes (no blocking issues)
- [ ] Warnings presented to user and acknowledged
- [ ] Ask user permission before creating new integration/live test scenarios
- [ ] If no automated verification is possible — provide manual verification steps

## Validation Gates

Run gate checks before advancing to the next pipeline stage. The canonical, complete criteria for every gate live in [SKILL.md → Validation Gates](../SKILL.md#validation-gates) — run that full checklist, never a from-memory subset. Review-phase supplements: the [Hallucination-Risk Heuristics](#hallucination-risk-heuristics) below extend the Specification → Plan gate; [Project Rules Compliance](#project-rules-compliance) extends the pre-commit code review.

## Hallucination-Risk Heuristics

Since the primary consumer of specifications is an LLM (AI agent), specs must be precise enough to prevent hallucinated implementations. During review, check each specification against these heuristics:

### Red Flags (block — must fix before advancing to plan)

| # | Heuristic | Example of violation | Fix |
|---|-----------|---------------------|-----|
| 1 | **Field without type** | `data: any` or `payload: object` | Specify concrete type or typed union |
| 2 | **Contract without error cases** | Operation lists only success path | Add Errors table with all failure conditions |
| 3 | **Prose instead of table for data structure** | "The entity has several fields including..." | Rewrite as a Fields table with types, defaults, constraints |
| 4 | **Vague constraint** | "should be reasonable", "within normal limits" | Replace with concrete bounds: `1..1000`, `max 255 chars` |
| 5 | **Missing default value** | Optional field without explicit default | Add default or document that null/empty is the default |
| 6 | **Unbounded collection** | `items: list` with no max size | Add max cardinality or pagination strategy |

### Yellow Flags (warn — acceptable if justified in the spec)

| # | Heuristic | Example | Acceptable when |
|---|-----------|---------|-----------------|
| 1 | **Generic enum** | `status: string` instead of enum values | Dynamic/extensible values documented |
| 2 | **No processing pseudocode** | Contract has input/output but no logic | Trivial CRUD operation |
| 3 | **Missing state transitions** | Entity has a `status` field but no transition diagram | Status is informational, not enforced |
| 4 | **Single error code** | Only one row in Errors table | Genuinely only one failure mode |
| 5 | **No invariants listed** | Data structure with no Invariants section | Entity is a simple value object |

### How to Apply

During the **Specification → Plan** gate check:
1. Scan every data structure section for red flags 1, 3, 4, 5, 6.
2. Scan every contract section for red flags 2.
3. For each yellow flag found, verify the justification exists in the spec.
4. If any red flag is found — **do not advance** to the plan. Fix the spec first.
5. Record findings in the review audit report.

### Precision Test

Ask yourself: *"If I gave this spec to a different LLM with no access to the codebase, would it generate the same data structures and contracts?"* If the answer is "probably not" — the spec is too vague.

## Project Rules Compliance

When `.dev_flow/rules/` exists, review new or modified code against the project rules.

### Rules Compliance Check

During the pre-commit code review of new or modified code:

1. Read `.dev_flow/rules/_index.yaml` to identify applicable rule categories.
2. For each category relevant to the changed files, read the rules document.
3. Check new code against rules by severity:
   - **must** rules — violation **blocks** advancement. Fix before proceeding.
   - **should** rules — violation triggers **warning**. Acceptable if justified in the review.
   - **prefer** rules — **informational**. Note deviation, no action required.
4. Record findings in the review report:

| Rule | Severity | File:Line | Status | Notes |
|------|----------|-----------|--------|-------|
| {RuleName} | must/should/prefer | path:line | pass/fail/waived | {justification if waived} |

### When Rules Are Updated

Rules documents in `.dev_flow/rules/` are living documents. They can be updated:
- During **onboard** — initial extraction from existing code.
- During **implement** — when a new pattern is established or discovered.
- During **review** — when a reviewer identifies an undocumented convention.
- By **user request** — explicit instruction to add, modify, or remove a rule.

When updating rules:
1. Edit the relevant `.dev_flow/rules/{category}.md` file.
2. Add the `Source` metadata: `onboard analysis | manual | phase observation`.
3. Update `.dev_flow/rules/_index.yaml` if a new rule was added.
4. Existing code is NOT required to be retroactively fixed.

**Reflection.** A reviewer finding that "this convention keeps being violated" is the review phase's [Experience Capture](../references/experience-capture.md) harvest: write it as a rule automatically through the [rule phase](rule.md) (default `should`; a would-be `must` or a contradiction routes to independent review first; the developer sees it in the commit diff), rather than an ad-hoc note that evaporates with the review's clean context.

## Conflict Resolution Protocol

All active concepts and specifications MUST be consistent with each other.

### Detecting Conflicts

A conflict exists when:
- Two concepts define overlapping responsibilities for the same domain area.
- A specification contradicts another specification it depends on.
- A new concept requires changes to an existing concept that others depend on.

When the conflict is not doc ↔ doc but **evidence ↔ doc** — review findings show a spec'd contract is ambiguous or cannot hold in reality — that is an upstream defect, not a resolution-priority question: route it through [Upstream Escalation](../references/escalation.md).

To enumerate a document's dependents and code bindings when checking a conflict or a deprecation, run the [Impact Walk](../references/impact.md).

### Resolution Process

| Step | Action |
|------|--------|
| 1 | Identify all conflicting documents and list specific contradictions. |
| 2 | Determine priority (newer requirement wins, unless older is load-bearing). |
| 3 | Update the lower-priority document to align. |
| 4 | Propagate changes to all dependent documents. |
| 5 | Verify no new conflicts were introduced. |

**Note:** Documents with `Status: draft` or `Status: deprecated` are excluded from conflict checks.

### Changelog Audit

During review, verify that all concepts, specifications, and plans have up-to-date Changelog tables reflecting significant changes since their creation.

## Deprecation & Removal

### Deprecating a Document

1. Set `Status: deprecated` in metadata.
2. Add `Deprecated-reason` field:
   ```
   > **Status:** deprecated
   > **Deprecated-reason:** Replaced by [C_XXX_V2](./new.concept.md)
   ```
3. Update `_index.md` to reflect deprecated status.
4. Update all `Used by` references — dependents should migrate.

### Behavior of Draft and Deprecated Documents

- NOT automatically updated when related active documents change.
- Ignored during conflict resolution.
- Content is informational and may be outdated.

### Removal Policy

A deprecated or draft document can be deleted when:
- Its `Updated` date is older than 3 months, AND
- No working code references its traceable IDs.

Before deleting:
1. Search codebase for references to the document's IDs.
2. Confirm no active documents list it in `Depends on`.
3. Remove its entry from `_index.md`.

## Document Index Maintenance

When `docs/` has more than 5 documents, maintain `_index.md`:
- One line per document — name and short description only.
- Sorted alphabetically by code.
- Updated every time you add, remove, or rename a document.
- Include `Status` column to identify deprecated or draft documents.

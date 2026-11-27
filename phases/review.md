# Phase 6: Review & Validation

## Contents

- [Purpose](#purpose) — scope of the phase: gates, document conflicts, deprecation, doc-code alignment, pre-commit review
- [Start Checklist](#start-checklist) — before a round: its snapshot and scope, the `Stage` row, the round mode, the level with its reason
- [Pre-Commit Code Review](#pre-commit-code-review) — clean-context reviewer, tripwires, round scoping/mode, checks table, materiality, report format
- [After Review: Verify Phase](#after-review-verify-phase) — hand-off to verify before commit; Review -> Verify gate checklist
- [Validation Gates](#validation-gates) — where each gate's criteria live, and this phase's supplements
- [Hallucination-Risk Heuristics](#hallucination-risk-heuristics) — red/yellow flag tables for specs, application steps, precision test
- [Project Rules Compliance](#project-rules-compliance) — rules activation protocol, severity handling, findings table, rule-update procedure
- [Conflict Resolution Protocol](#conflict-resolution-protocol) — conflict detection criteria, resolution steps table, changelog audit
- [Deprecation & Removal](#deprecation--removal) — deprecating a document, draft/deprecated behavior, removal policy
- [Document Index Maintenance](#document-index-maintenance) — `_index.md` rules when docs/ exceeds 5 documents

## Purpose

Validate pipeline gates, detect conflicts between documents, manage deprecation, ensure the documentation-code alignment is maintained, and perform pre-commit code review of changes.

## Start Checklist

Before the first action: the [Phase start checklist](../SKILL.md#phase-start-checklist), then:

- [ ] **Round snapshot and scope** — this round's snapshot, taken before the reviewer starts, and its scope: round 1 or a full round (with the reason), or delta ∪ carry-over ([step 0](#pre-commit-review-procedure))
- [ ] **`Stage` row** — checked against the facts; rewritten when they make the stage `production` ([step 0](#pre-commit-review-procedure))
- [ ] **Round mode** — `confirm` or `full`; round 1 always `full` ([step 0b](#pre-commit-review-procedure))
- [ ] **Review level** — the round's level with its reason; a level the developer named is the `Review level` row of the task file ([step 1](#pre-commit-review-procedure))

## Pre-Commit Code Review

**Before any commit**, a code review of the changes MUST be performed by a **subagent with a clean context** — it sees the diff, the spec, and the code the diff reaches — never the implementer's accumulated assumptions.

This clean-context review also **realizes the `sampled-verifier` tier** of [Application Enforcement](../references/application-enforcement.md).

A checkable guard on the reviewer itself: **2+ consecutive substantive diffs** (non-trivial change class) reviewed at level `standard` or `deep` with **zero actionable findings** (blocking/should-fix, not nits) is a `verifier-rubber-stamp` tripwire; a `formal` or `quick` review in between neither counts nor breaks the run. Surface it to the user; switch model if the harness has one.

Findings carry a computed **materiality**, a recurring non-`must` finding trips `review-non-convergence` and leaves the loop as a `contested` todo, and a repeat round reads only the fixes — the delta from the snapshot of the previous round — plus the open findings. A `full` round runs at a [review level](../references/review-convergence.md#review-level--how-deep-a-round-looks) — `formal`, `quick`, `standard` or `deep` — that sets which checks the reviewer opens, and from round 3 a finding no earlier round raised blocks only when it is a `must` or [real-risk security finding](../references/security-scope.md#risk--computed-not-judged) or comes from a check a raise opened in that round ([round-3 cap](../references/review-convergence.md#finding-materiality--computed-not-judged)). Procedure: **[Review Convergence](../references/review-convergence.md)**.

### Pre-Commit Review Procedure

0. **Scope the round** ([Incremental round scope](../references/review-convergence.md#incremental-round-scope)). Take this round's snapshot now, before the reviewer starts. Round 1 reads `git diff HEAD <snapshot>`, untracked files included (no commit yet: `git diff $(git hash-object -t tree /dev/null) <snapshot>`). A later round reads only **delta ∪ carry-over**: the delta is `git diff <baseline> <snapshot>` — the baseline the previous round wrote against this round's snapshot; a raised round also reads the whole diff for the checks the raise opens (step 1). The one full reason is no reachable baseline. Name the scope — and the reason for a full round — in the report. Check the `Stage` row against the facts and rewrite it when they make the stage `production` ([Project stage](../references/delivery-focus.md#project-stage)); steps 0b, 1 and 3 read the rewritten row.

0b. **Select the round mode** (repeat rounds only — [Verification Economy](../references/verification-economy.md)). A contained fix of a prescribed finding that is neither `must` nor a [real-risk security finding](../references/security-scope.md#risk--computed-not-judged) runs as `confirm`: check the diff against the prescribed fix and that every task-set test that was green is still green ([Test Economy](../references/test-economy.md#task-test-set)) — by its run record after the fix, or the record from before it when its [Reuse](../references/run-economy.md#reuse) verdict is `reuse` — then skip to step 5. Everything else, and round 1 always, runs `full` — continue to step 1. A `confirm` round that does not confirm replays as `full` from step 1.

1. **Choose the round's level and launch a reviewer subagent** (role: [reviewer.ai.md](../roles/reviewer.ai.md)). Choose the level by [Review level → Choosing](../references/review-convergence.md#review-level--how-deep-a-round-looks), raised by the [Raise](../references/review-convergence.md#raise) rule on a repeat round. A level the developer named for the task is the `Review level` row of the task file; a level the developer names — in the request or at any later moment — is written to the row at once, with the quote; one the row does not hold yet at this step → write it first. The brief opens with `Skill root: <absolute path of the skill>` and `Read first: roles/reviewer.ai.md — blocks: rules, pre_commit_review, convergence`, then the `Role style:` line of the phase and `Review level: <level> — <reason>`; a project overlay listed in `.dev_flow/roles/_index.yaml` is added after the base role ([SKILL.md → Subagents](../SKILL.md#subagents)). Besides these lines, the `Shell:` and `Cut limit:` lines and the friction line of [Delegation for Focus → How](../references/delegation.md#how), the clean context contains only:
   - Round 1 and a full round: `git diff HEAD <snapshot>` of this round, untracked files included. A delta round: the delta diff and the carry-over findings, each with its identity, severity and prescribed fix in full — never the whole diff; the brief states that a finding is raised only in the delta or where the delta breaks unchanged code, except a `must` defect or a real-risk security finding seen anywhere ([Incremental round scope](../references/review-convergence.md#incremental-round-scope)). A raised round adds, for the checks the raise opens, the whole diff — `git diff HEAD <snapshot>` of this round ([Raise](../references/review-convergence.md#raise))
   - The effective criticality of the changed area ([Functional criticality](../references/review-convergence.md#functional-criticality--declared-upstream-read-downstream)) — read by the main agent from the owning concept and spec; `unstated` when none declares it
   - The stage written in the task file — the Security check opens by it and a security finding's risk reads it ([Security Scope](../references/security-scope.md#security-work-by-phase-and-stage)); kept at every level
   - While the stage written in the task file is `production`: every open deferred security finding with the trigger "before the first external exposure" — in `.dev_flow/todos/`, the plan backlogs and the backlogs of the audit plans (every `*.plan.md` under `.dev_flow/audit/`) — as a carry-over finding (step 3)
   - The relevant specification (`*.sp.md`) for contract verification
   - The relevant plan (`*.plan.md`) for completeness verification
   - Project rules (`.dev_flow/rules/`) if they exist — binding
   - Project `check` tools (`.dev_flow/tools/_index.yaml`) whose `applies_to` matches the changed files and the `review` phase — the reviewer runs each on the diff after its [trust check](../references/project-tools.md#trust-check); its findings are findings of the rule that names it in a `Check:` line, under that rule's severity (`should` when no rule names it), never a verdict of their own ([Checkability rung](../references/experience-capture.md#checkability-rung))
   - Relevant project skills (`.dev_flow/skills/`) for the changed area, if they exist
   - SOLID architecture reference (`references/solid-architecture.md`)
   - `docs/_framework.md` (the architectural map) if present
   - `docs/_index.md` and `docs/_glossary.md` when present
   - Read access to the codebase — the reviewer itself greps the callers of every symbol whose signature, return value, or behaviour the diff changes; never pass the author's list ([Symbol reach](../references/impact.md#symbol-reach))

   The level trims the list to what its checks need: a `formal` brief leaves out the specification, the plan, the skills, the SOLID reference, `docs/_framework.md`, `docs/_index.md`, `docs/_glossary.md` and the codebase read access — except the reads the Security check's risk argument needs; a `quick` brief leaves out the SOLID reference, `docs/_framework.md`, `docs/_index.md` and `docs/_glossary.md`; a `deep` brief adds the area's concept and the inputs of the [Impact Walk](../references/impact.md#input).

   **Pass the artifact + contract only — never the author's conclusion or self-assessment; frame the review adversarially** ("find what violates the spec/rules", not "is this good?"). Realizes the `sampled-verifier` contract ([Application Enforcement](../references/application-enforcement.md)).

2. **The reviewer subagent checks:**

   | Check | Description | Finding severity | Opens at level |
   |-------|-------------|------------------|----------------|
   | Spec compliance | Code implements all spec contracts, error cases, invariants | `must` | `quick` and up — at `quick`, the contracts the diff touches |
   | Plan completeness | All plan tasks for the current phase are addressed | `must` | `quick` and up |
   | Rules compliance | New code follows `.dev_flow/rules/` | the rule's own severity | every level |
   | Skill pitfalls | Change doesn't reintroduce a pitfall documented in a loaded skill | `should` (`must` if also a `must` rule) | `quick` and up |
   | SOLID compliance | Code structure follows SOLID and pluggability principles ([reference](../references/solid-architecture.md)) unless overridden by project rules | `should` | `standard` and up |
   | Code reuse | New code reuses existing functions/classes instead of re-implementing them; a new reuse seam has a real consumer ([reference](../references/code-reuse.md)); look-alikes that change for different reasons are not flagged | `should` | `standard` and up |
   | No regressions | Changes don't break existing functionality — checked against the callers of every changed symbol, read outside the diff | `must` | `quick` and up |
   | Docs in sync | The documents the change reaches describe what the diff now does ([docs sync](propagate.md#docs-sync-at-task-end)) — at `standard`, the spec and plan of the brief, `docs/_index.md`, `docs/_glossary.md` and `docs/_framework.md`; at `deep`, also the area's concept and the dependents of the Impact Walk radius; a needed edit left out of the diff — or, when `docs/` is ignored by git, out of the working tree — is a finding | `should` | `standard` and up |
   | No leftover artifacts | No debug code, TODOs, commented-out blocks | `should` | every level |
   | Test economy | Each new permanent test has a keep reason in the test report; no temporary test in the diff; no test deleted outside `audit tests` while the code it checked stays ([Test Economy](../references/test-economy.md#review-check)) | `should` | `quick` and up |
   | Promoted prototype | The task promotes a prototype → a file of the diff identical to its counterpart in the prototype place the spike names has the `keep` mark in the plan; an unmarked one is a finding ([research](research.md#prototype-in-the-project)) | `must` | `quick` and up |
   | Code quality | Naming, structure, readability follow project conventions | `prefer` | `standard` and up |
   | Security | By [Security Scope](../references/security-scope.md#security-work-by-phase-and-stage): in code, a vulnerability — injection, exposure, a broken item of the [secure-coding baseline](../references/security-scope.md#secure-coding-baseline); in a concept, spec or plan, only the security mechanisms the document decides — a hypothetical vulnerability of code not yet written is no finding. A secret counts only where secrets never live ([Proportionate Caution](../references/proportionate-caution.md#secrets)). Each finding carries its [risk argument](../references/security-scope.md#risk-argument) — actor, entry, chain, precondition, exposure, impact; a measure it prescribes never restricts a function ([Protect, never restrict](../references/security-scope.md#protect-never-restrict)) | `must`; `should` for a finding of risk `deferred` or `hypothetical` | every level — at `formal`, the diff lines and the reads their risk argument needs; not opened while the stage written in the task file is `prototype`, except a `secret-exposure` of a real secret |

   The severity column is the finding's **severity**, on the same `must` / `should` / `prefer` axis as project rules. It is the reviewer's input to the materiality computation in step 3 — not the verdict. A check's severity is a ceiling per finding, not a floor: a reviewer states a lower severity when the concrete finding warrants it.

   The level column says from which [review level](../references/review-convergence.md#review-level--how-deep-a-round-looks) of the round a check opens; `deep` adds its four reads to the checks of `standard`. A check the level does not open is listed in the report's `Unobserved` section in one line — `not opened at level <level>: <checks>` — never omitted; a round at any level reports a `must` defect or a real-risk security finding it sees, outside its checks or the diff included. At the stage `prototype` the Security check is listed there as `not opened at stage prototype: Security`; a `secret-exposure` of a real secret is reported at every stage.

3. **Classify each finding** ([Review Convergence](../references/review-convergence.md)). The reviewer assigns `severity` (`must` / `should` / `prefer`); **materiality is computed, never assigned** — from `severity`, finding type, and the effective `Criticality` declared by the owning concept/spec. `blocking` spawns the next round; `advisory` is reported only; `deferrable` leaves the loop at once. In round 3 and later, a `blocking` verdict of the table for an identity no earlier round of the task raised becomes `advisory`, except a finding of a check the raise opened in this round ([Round-3 cap](../references/review-convergence.md#finding-materiality--computed-not-judged)); the main agent applies it to the report, reading the earlier identities from the carry-over column of the task file's Review Rounds table. A security finding takes its materiality from its [risk](../references/security-scope.md#risk--computed-not-judged) ([override 1](../references/review-convergence.md#finding-materiality--computed-not-judged)): a `must` finding and a real-risk security finding are always `blocking` and never leave the loop; a security finding of risk `deferred` is filed as a `deferred` todo and appends a `hardening` row; one of risk `hypothetical` is a warning with its recommendation and the field it lacks. While the stage written in the task file is `production`, every open deferred security finding with the trigger "before the first external exposure" — in `.dev_flow/todos/`, the plan backlogs and the backlogs of the audit plans (every `*.plan.md` under `.dev_flow/audit/`) — is `blocking` in this review, inside or outside the diff ([The first external exposure](../references/delivery-focus.md#project-stage)); step 1 passes them to the reviewer as carry-over. When the review closes the finding of such a record — or the developer answers `build` on its `hardening` row — mark the record `promoted` with this task's id (a plan backlog item: append `— promoted: task_<ID>`), in the task's commit.

4. **Review result:**
   - **Pass** — proceed to commit approval; on the yes at the task's last commit, [close the task](status.md#on-task-completion) before `git commit`.
   - **Fail (blocking)** — fix issues, re-run the task-set tests that cover the fix ([When to run](../references/test-economy.md#when-to-run)) and, when the fix edited code, the checks the fast variant turned off on the modules it touched ([Run Economy → Fast variant](../references/run-economy.md#fast-variant)), and, when the fix changes what a document describes, the [docs sync](propagate.md#docs-sync-at-task-end) again for that document, then re-review. Select the repeat round's mode first: a contained fix of a prescribed finding that is neither `must` nor a real-risk security finding is a `confirm` round in the main context, everything else spawns the clean-context reviewer, at the level step 1 chooses with the [Raise](../references/review-convergence.md#raise) rule. A `confirm` round that does not confirm replays as `full` — it never closes a finding it did not confirm. See [Verification Economy](../references/verification-economy.md). A non-`must` blocking finding that recurs unresolved into a second round trips `review-non-convergence` and routes to a `contested` todo instead of a third round. A finding with a `bypassed_list` is fixed by changing the path, not by a list member alone unless the owning document records the list as the defence; no path → upstream escalation ([List bypass](../references/review-convergence.md#list-bypass--fix-the-path-not-the-list)). When the only fix of a real-risk security finding restricts a function the request, the developer, a spec contract or a project rule names, ask the developer by [Protect, never restrict](../references/security-scope.md#protect-never-restrict) before any fix; an accepted risk is a decision record in the owning document — the task file when there is none — and closes the finding.
   - **Warnings only** — present warnings to the user, proceed if user approves. An advisory applied after the review is checked as a `confirm` round ([Verification Economy → An applied advisory is confirmed](../references/verification-economy.md#discriminator--re-verifying-a-known-fix)).

5. **Report format:**

   ```
   ## Pre-Commit Review — round {N}{, level {formal|quick|standard|deep} — {reason}}, scope {full|delta}{, raised checks full}{, baseline {tree-hash}}, mode {full|confirm}{, reason: {full reason}}

   **Result:** PASS / FAIL / WARNINGS

   ### Blocking Issues
   - [ ] {issue description} — {file:line} — check {check}, type {type}, severity {must|should}, criticality {value|unstated}, recurrence {n}{, bypassed list {name — file:line}}{, risk real — actor {actor}, entry {file:line}, chain {file:line → …}, precondition {precondition|none}, exposure {external|local}, impact {impact}}{ — fix: {the one concrete fix}}

   ### Warnings
   - {warning description} — {file:line} — advisory under criticality {value|unstated}
   - {security finding} — {file:line} — risk hypothetical: {the field it lacks} — recommendation: {what would protect, never a restriction}

   ### Routed to todo
   - {TD_id} — {issue} — contested: {why it did not converge; both positions recorded} | deferred: risk deferred, exposure external, stage {stage}, trigger "before the first external exposure"

   ### Unobserved
   - {check the reviewer could not evaluate} — {what was missing: no spec for the area, no test result, no runtime access}
   - {check the reviewer did not open} — {the absent entry condition: no signal is true for this file}
   - not opened at level {level}: {checks}
   - not opened at stage prototype: Security

   ### Summary
   {1-2 sentence summary of the review}
   ```

   A check the reviewer could not evaluate, or did not open for a missing entry condition ([Verification Economy](../references/verification-economy.md)), is listed as `unobserved` with the absent fact — it is neither a pass nor a warning, and never omitted. Every routed record is named; a silently deferred finding is a report-ceiling violation. The summary claims no more than the evidence supports. See [Evidence Discipline](../references/evidence-discipline.md).

6. **Close the round.** Write `baseline` (this round's snapshot tree hash, taken in step 0 — [Incremental round scope](../references/review-convergence.md#incremental-round-scope)), the round mode, the level of a `full` round with its reason, and `carry_over` (open blocking findings: identity + severity + recurrence + prescribed fix in full + bypassed list) into the task file, so the next round scopes, selects its mode and raises its level from them. For each project rule the round's Rules Compliance table marks `fail` (finding type `rule:<Name>`), append a `recurred` observation to the rule's ledger entry — once per rule per task; a later round of the same task extends that observation's note ([Experience Capture](../references/experience-capture.md#applying-experience--auto-apply-through-structural-gates)). A failed append skips the observation, never the round.

## After Review: Verify Phase

After the review passes, proceed to the [Verify phase](verify.md) for regression, integration, and live testing. Do NOT commit directly after review — verification must pass first.

### Gate Check (Review -> Verify)

- [ ] Pre-commit review by a clean-context subagent passes (no blocking issues)
- [ ] Warnings presented to user and acknowledged
- [ ] Ask user permission before creating new integration/live test scenarios
- [ ] If no automated verification is possible — provide manual verification steps

## Validation Gates

Run gate checks before advancing to the next pipeline stage. Each gate's complete criteria live in a phase file; the table in [SKILL.md → Validation Gates](../SKILL.md#validation-gates) links each one — run that full checklist, never a from-memory subset. Review-phase supplements: the [Hallucination-Risk Heuristics](#hallucination-risk-heuristics) below extend the Specification → Plan gate; [Project Rules Compliance](#project-rules-compliance) extends the pre-commit code review.

## Hallucination-Risk Heuristics

During review, check each specification against these heuristics:

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

1. Read `.dev_flow/rules/_index.yaml` (by category block when it is above `full_read_limit`) and match category `applies_to` selectors against the changed files and the `review` phase (always-on categories included) — the [Knowledge Scaling](../references/knowledge-scaling.md) activation protocol.
2. Take the matched categories' `rules[]` from the index, filter by unit selector → the relevant set of directives; over `activation_budget` drops nothing and lowers no severity; read a rule's body (ranged by its anchor) only on a trigger — the directive does not decide the case, a violation is suspected or found, an example is needed, the rule itself is under edit.
3. Check new code against rules by severity:
   - **must** rules — violation **blocks** advancement. Fix before proceeding.
   - **should** rules — violation triggers **warning**. Acceptable if justified in the review.
   - **prefer** rules — **informational**. Note deviation, no action required.
4. Record findings in the review report:

| Rule | Severity | File:Line | Status | Notes |
|------|----------|-----------|--------|-------|
| {RuleId} | must/should/prefer | path:line | pass/fail/waived | {justification if waived} |

### When Rules Are Updated

Rules are living documents ([rule phase](rule.md)); a review finding is one of the moments they change.

When updating rules:
1. Edit the relevant `.dev_flow/rules/{category}.md` file in the unit form (directive in the heading, bounded body — [rule phase](rule.md)); a lesson from the review is a rule; skills form later from accumulated clusters ([Knowledge Scaling](../references/knowledge-scaling.md)).
2. Record `Provenance` (the review's task file) in the unit body.
3. Update the category's `## Contents` digest line and `.dev_flow/rules/_index.yaml` in the same edit.
4. Existing code is NOT required to be retroactively fixed.

**Reflection.** A reviewer finding that "this convention keeps being violated" is the review phase's [Experience Capture](../references/experience-capture.md) harvest: write it as a rule automatically through the [rule phase](rule.md) (default `should`; a would-be `must` or a contradiction routes to independent review first; the developer sees it in the commit diff); a `mechanical` class goes to a `check` tool instead, a `mixed` one to both ([Checkability rung](../references/experience-capture.md#checkability-rung)). A lesson an existing rule, skill, or tool already covers appends a `recurred` or `exercised` observation to that artifact's ledger entry instead of a new write — a rule's `fail` that step 6 already recorded is that observation, and a hand-rolled check a covering tool covers records a [bypass](../references/project-tools.md#use-accounting) in place of `recurred`, unless one is already recorded for this check (by a failed run or an earlier rung) — never a second ([Experience Capture](../references/experience-capture.md#applying-experience--auto-apply-through-structural-gates)). A gap escalated to a specification is routed by [Sweep gaps](../references/experience-capture.md#sweep-gaps).

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

During review, verify that concepts, specifications, and plans have Changelog entries that pass the [Docs Scaling](../references/docs-scaling.md) content filter.

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
- Its `Updated` date is older than `stale_after` ([Docs Scaling](../references/docs-scaling.md)), AND
- No working code references its traceable IDs.

Before deleting:
1. Give the document its [recovery point](../references/recovery-points.md#recovery-point).
2. Search codebase for references to the document's IDs.
3. Confirm no active documents list it in `Depends on`.
4. Remove its entry from `_index.md`.

## Document Index Maintenance

When `docs/` has more than 5 documents, maintain `_index.md`:
- One line per document — name and short description only.
- Sorted alphabetically by code.
- Updated in the same step that adds, removes, or renames a document, without asking — spikes and adoption pairs included ([Docs Scaling → Index router](../references/docs-scaling.md#index-router)).
- Include `Status` column to identify deprecated or draft documents.

# Phase 5: Testing (Functional)

## Purpose

After code changes, run the **task's functional tests** (unit + mock) to catch regressions quickly. This phase runs only the [task test set](../references/test-economy.md#task-test-set) — not the full test suite.

Integration and live tests are handled separately in the [Verify phase](verify.md), which runs after Review.

The `test` command can create new tests, edit existing ones, run tests, or execute complex test scenarios.

## Activation Condition

This phase activates **only when both conditions are met:**
1. The project has an existing test suite (e.g., `tests/` directory, test runner configured).
2. There are defined rules for running and validating tests (e.g., `pytest`, `jest`, CI config).

If neither exists — skip directly to Review (Phase 6: [review](review.md)), and record the skip as `unobserved` with the reason (no harness), never as a clean result — see [Evidence Discipline](../references/evidence-discipline.md).

## Command

```
/dev-flow test [target]
```

The target is optional. Without it, tests are determined from the current active context.

### Examples

```
/dev-flow test                          # Run tests for current active task
/dev-flow test auth module              # Test the auth module specifically
/dev-flow test add login validation     # Create permanent tests on request; mark the criterion `keep`
/dev-flow test update rate limiter      # Update existing rate limiter tests
```

Integration and live tests are **not** run here — they belong to the [Verify phase](verify.md) (`/dev-flow verify integration …`, `/dev-flow verify live …`).

## Role Responsible

This phase is handled by **Tester**: [roles/tester.ai.md](../roles/tester.ai.md). A delegated run: the brief carries `Skill root: <absolute path of the skill>` and `Read first: roles/tester.ai.md` ([SKILL.md → Subagents](../SKILL.md#subagents)).

## Test Categories (This Phase)

This phase covers **functional tests only** — fast, isolated checks of the changed code:

| Level | Type | Purpose | When to run |
|-------|------|---------|-------------|
| 1 | **Unit tests** | Verify individual functions/methods in isolation | After a change to the code they cover |
| 2 | **Functional / Mock tests** | Verify behavior with mocked dependencies | When code interacts with external services or complex subsystems |

**Rule:** Run only the task's tests, at the moments of [Test Economy](../references/test-economy.md#when-to-run). Do not run the full test suite — that is the job of regression testing in the Verify phase.

## Rules

1. **Check every verification criterion the change touches** — contracts, error cases, invariants. A criterion that is not checked is `unobserved` with its reason.
2. **A permanent test needs a [keep reason](../references/test-economy.md#keep-reasons).** A criterion marked `keep` in the spec's `Test` column always gets a permanent test.
3. **Check trivial behaviour with a [temporary test](../references/test-economy.md#temporary-tests).** It ends with the task.
4. **Use the project's existing test conventions.** Do not introduce a new test framework or test structure — follow what the project already uses.
5. **Reference traceable IDs in test names or docstrings:**
   ```python
   def test_acquire_token_rate_limited():
       """[SP_RLM_02_01] AcquireToken returns wait_seconds when bucket is empty."""
   ```

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills for the tested functionality — their pitfalls and edge cases MUST be checked. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load testing rules (naming, assertion patterns, mocking); new tests MUST comply. See [rule phase](rule.md).

**Knowledge activation (per-burst).** Re-trigger the gate above at the moment of action — before a test-writing or test-run burst, re-surface the applicable rules/skills (esp. a procedure with a sanctioned runner) with a pointer-only Pre-Action Marker; for a high-stakes procedure prefer a deterministic tripwire (non-model signal: exit code / captured command) or a sampled cross-model verifier where the runtime allows. Self-attestation is never the control. See [Application Enforcement](../references/application-enforcement.md).

## Gate Check (Code -> Test)

Before running tests, verify:
- [ ] Every verification criterion the change touches is checked; an unchecked one is `unobserved` with its reason
- [ ] A criterion marked `keep` has a permanent test; every other new permanent test names its keep reason
- [ ] Trivial checks leave no permanent test
- [ ] Only the task's tests are run (unit + mock) — never the full suite

## Testing Workflow

```
1. Identify affected functionality from the code changes — read the implemented
   plan phase's `Verify:` field first: it names the spec Verification Criteria
   (SP_XXX_05_*) and acceptance checks for exactly this phase (the ready checklist;
   no re-derivation from the whole spec)
2. Find existing unit and mock tests covering the changed code
3. Choose the form of each needed check by Test Economy
   (../references/test-economy.md#choosing-the-form): a permanent test only with
   a keep reason, trivial behaviour as a temporary test; no permission needed
4. Run the task test set once (unit + mock) — never the full suite
5. If any test fails:
   a. Analyze the failure — a code bug, a test bug, or a spec problem?
   b. Code or test bug → fix the root cause (code must satisfy the spec,
      not the other way around)
   c. Spec problem — two defensible readings of a contract, or the contract
      contradicts observed reality → stop and escalate upstream instead of
      guessing; see Upstream Escalation (../references/escalation.md)
   d. Re-run the failed tests
6. All functional tests pass → delete in-tree temporary tests → proceed to Review phase
```

**Note:** Do NOT run integration or live tests at this stage. Those are handled in the [Verify phase](verify.md) after Review passes.

### Creating and Updating Tests

When the `test` command targets test creation or modification:

1. **Read the specification** — identify contracts, error cases, invariants.
2. **Read existing tests** — understand conventions, helpers, fixtures.
3. **Write or update tests** following the project's patterns:
   - Choose the form of each check by [Test Economy](../references/test-economy.md#choosing-the-form); a permanent test only with a keep reason.
   - Group permanent tests by contract or entity.
   - Check the happy path, error cases, and edge cases.
   - Include traceable ID references.
4. **Run the new/updated tests** to verify they pass.

## Test Result Reporting

After running tests, report:

| Item | Detail |
|------|--------|
| Tests run | Total count by level |
| Passed | Count |
| Failed | Count + failure details |
| Skipped | Count + reason |
| Unobserved | Contracts/criteria this environment could not verify + the missing observation boundary (no harness / no fixture / no service) |
| Permanent tests added | Each test with its keep reason and traceable ID; the commit carries a `Keep-Test` trailer per test |
| Temporary checks | Number of temporary tests and the criteria they checked |
| Tests updated | List with change description |

The report claims no more than the evidence supports — "covered" only where a test actually ran; an unverifiable criterion is `unobserved` with a named reason, never silence. See [Evidence Discipline](../references/evidence-discipline.md).

## Gate Check (Test -> Review)

Before the review starts, verify:
- [ ] All relevant functional tests pass
- [ ] No regressions in changed code area

## Reflection

At the phase boundary, run the [Transition Checkpoint](../references/experience-capture.md): if writing tests surfaced a recurring assertion/mocking pattern or a coverage gap worth a convention, write a testing rule (`.dev_flow/rules/testing.md`); if it produced a non-trivial fixture/harness technique, write/update a skill — never a fact about the developer or the machine, which belongs to `.dev_flow/profile/` under its consent regime ([Adaptation Profile](../references/adaptation-profile.md)). Auto-apply through the structural [rule](rule.md)/[skill](skill.md) gate — no permission prompt; never auto-write a `must` (route those + contradictions to independent review); every write is visible in the commit diff. A lesson an existing rule, skill, or tool already covers appends a `recurred` or `exercised` observation to that artifact's ledger entry instead of a new write — a hand-rolled check a covering tool covers records a [bypass](../references/project-tools.md#use-accounting) in place of `recurred`, unless one is already recorded for this check (by a failed run or an earlier rung) — never a second ([Experience Capture](../references/experience-capture.md#applying-experience--auto-apply-through-structural-gates)). See **[Experience Capture](../references/experience-capture.md)**.

## Anti-Patterns

- Writing tests that verify implementation details instead of spec contracts
- Skipping error-case checks ("happy path only")
- Modifying the spec to match failing code instead of fixing the code (a genuine spec defect goes through [Upstream Escalation](../references/escalation.md), not a silent edit)
- Introducing a new test framework when the project already has one
- Writing tests without traceable ID references
- Running the full test suite outside Verify, or a test again when the code it covers did not change
- Running integration/live tests in this phase (those belong in Verify)
- Asking permission for a unit or mock test
- A permanent test without a keep reason
- Deleting a failing test to get a pass

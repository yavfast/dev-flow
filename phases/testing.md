# Phase 5: Testing (Functional)

## Purpose

After code changes, run **functional tests** (unit + mock) covering the changed code to catch regressions quickly. This phase runs only tests relevant to the modified area — not the full test suite.

Integration and live tests are handled separately in the [Verify phase](verify.md), which runs after Review.

The `test` command is flexible — it can create new tests, edit existing ones, run tests, or execute complex test scenarios.

## Activation Condition

This phase activates **only when both conditions are met:**
1. The project has an existing test suite (e.g., `tests/` directory, test runner configured).
2. There are defined rules for running and validating tests (e.g., `pytest`, `jest`, CI config).

If neither exists — skip directly to Review (Phase 6: [review](review.md)).

## Command

```
/dev-flow test [target]
```

The target is optional. Without it, tests are determined from the current active context.

### Examples

```
/dev-flow test                          # Run tests for current active task
/dev-flow test auth module              # Test the auth module specifically
/dev-flow test add login validation     # Create tests for login validation
/dev-flow test update rate limiter      # Update existing rate limiter tests
```

Integration and live tests are **not** run here — they belong to the [Verify phase](verify.md) (`/dev-flow verify integration …`, `/dev-flow verify live …`).

## Test Categories (This Phase)

This phase covers **functional tests only** — fast, isolated checks of the changed code:

| Level | Type | Purpose | When to run |
|-------|------|---------|-------------|
| 1 | **Unit tests** | Verify individual functions/methods in isolation | Always — after any code change |
| 2 | **Functional / Mock tests** | Verify behavior with mocked dependencies | When code interacts with external services or complex subsystems |

**Integration and live tests** (levels 3-4) are handled in the [Verify phase](verify.md), which runs after Review.

**Rule:** Run only the tests that cover the changed code. Do not run the full test suite — that is the job of regression testing in the Verify phase.

## Rules

1. **Test every spec contract.** Each operation from the specification's Contracts section must have at least one test verifying its input/output behavior.
2. **Test all error cases.** Every row in the spec's Errors tables must have a corresponding test that triggers the condition and verifies the expected error code/behavior.
3. **Test invariants.** Each invariant listed in the spec's Data Structures section must be verified (e.g., immutability constraints, state transition rules).
4. **Use the project's existing test conventions.** Do not introduce a new test framework or test structure — follow what the project already uses.
5. **Reference traceable IDs in test names or docstrings:**
   ```python
   def test_acquire_token_rate_limited():
       """[SP_RLM_02_01] AcquireToken returns wait_seconds when bucket is empty."""
   ```

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills for the tested functionality — their pitfalls and edge cases MUST be covered by the tests. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml` and load testing rules (naming, assertion patterns, mocking); new tests MUST comply. See [rule phase](rule.md).

**Knowledge activation (per-burst).** Re-trigger the gate above at the moment of action — before a test-writing or test-run burst, re-surface the applicable rules/skills (esp. a procedure with a sanctioned runner) with a pointer-only Pre-Action Marker; for a high-stakes procedure prefer a deterministic tripwire (non-model signal: exit code / captured command) or a sampled cross-model verifier where the runtime allows. Self-attestation is never the control. See [Application Enforcement](../references/application-enforcement.md).

## Gate Check (Code -> Test)

Before running tests, verify:
- [ ] All spec contracts have corresponding test cases
- [ ] All error cases from spec's Errors tables are tested
- [ ] All invariants from spec are verified in tests
- [ ] Tests use the project's existing conventions and runners

## Testing Workflow

```
1. Identify affected functionality from the code changes — read the implemented
   plan phase's `Verify:` field first: it names the spec Verification Criteria
   (SP_XXX_05_*) and acceptance checks for exactly this phase (the ready checklist;
   no re-derivation from the whole spec)
2. Find existing unit and mock tests covering the changed code
3. Check if new functional test cases (unit/mock) are needed — create them without
   asking (they are safe; see Anti-Patterns)
4. Run only the relevant functional tests (unit + mock)
5. If any test fails:
   a. Analyze the failure — a code bug, a test bug, or a spec problem?
   b. Code or test bug → fix the root cause (code must satisfy the spec,
      not the other way around)
   c. Spec problem — two defensible readings of a contract, or the contract
      contradicts observed reality → stop and escalate upstream instead of
      guessing; see Upstream Escalation (../references/escalation.md)
   d. Re-run the failed tests
6. All functional tests pass → proceed to Review phase
```

**Note:** Do NOT run integration or live tests at this stage. Those are handled in the [Verify phase](verify.md) after Review passes.

### Creating and Updating Tests

When the `test` command targets test creation or modification:

1. **Read the specification** — identify contracts, error cases, invariants.
2. **Read existing tests** — understand conventions, helpers, fixtures.
3. **Write or update tests** following the project's patterns:
   - Mirror spec structure: one test group per contract/entity.
   - Cover happy path, error cases, and edge cases.
   - Include traceable ID references.
5. **Run the new/updated tests** to verify they pass.

## Test Result Reporting

After running tests, report:

| Item | Detail |
|------|--------|
| Tests run | Total count by level |
| Passed | Count |
| Failed | Count + failure details |
| Skipped | Count + reason |
| New tests added | List with traceable IDs |
| Tests updated | List with change description |

## Reflection

At the phase boundary, run the [Transition Checkpoint](../references/experience-capture.md): if writing tests surfaced a recurring assertion/mocking pattern or a coverage gap worth a convention, write a testing rule (`.dev_flow/rules/testing.md`); if it produced a non-trivial fixture/harness technique, write/update a skill. Auto-apply through the structural [rule](rule.md)/[skill](skill.md) gate — no permission prompt; never auto-write a `must` (route those + contradictions to independent review); every write is visible in the commit diff. See **[Experience Capture](../references/experience-capture.md)**.

## Anti-Patterns

- Writing tests that verify implementation details instead of spec contracts
- Skipping error case tests ("happy path only")
- Modifying the spec to match failing code instead of fixing the code (a genuine spec defect goes through [Upstream Escalation](../references/escalation.md), not a silent edit)
- Introducing a new test framework when the project already has one
- Writing tests without traceable ID references
- Running the full test suite when only a small area changed (wastes time)
- Running integration/live tests in this phase (those belong in Verify)
- Asking permission for unit/mock tests (they are safe — just create them)

# Phase 5: Testing

## Purpose

After code changes, verify that the affected functionality works correctly.
This includes adding or updating tests, running them across multiple levels,
and fixing any issues discovered.

The `test` command is flexible — it can create new tests, edit existing ones,
run tests, or execute complex test scenarios.

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
/dev-flow test integration sync         # Run integration tests for sync
/dev-flow test live api endpoints       # Run live tests against real API
```

## Test Categories

Tests are organized by level. After code changes, identify which levels are affected
and run them in order (fastest first):

| Level | Type | Purpose | When to run |
|-------|------|---------|-------------|
| 1 | **Unit tests** | Verify individual functions/methods in isolation | Always — after any code change |
| 2 | **Functional / Mock tests** | Verify behavior with mocked dependencies | When code interacts with external services or complex subsystems |
| 3 | **Integration tests** | Verify interaction between real components | When contracts between modules change |
| 4 | **Live tests** | Verify against real external services/APIs | When integration points or configurations change |

**Rule:** Run the minimum set of tests that covers the affected functionality.
Do not skip a level if the changes could affect it.

## Rules

1. **Test every spec contract.** Each operation from the specification's Contracts section
   must have at least one test verifying its input/output behavior.
2. **Test all error cases.** Every row in the spec's Errors tables must have a corresponding
   test that triggers the condition and verifies the expected error code/behavior.
3. **Test invariants.** Each invariant listed in the spec's Data Structures section must be
   verified (e.g., immutability constraints, state transition rules).
4. **Use the project's existing test conventions.** Do not introduce a new test framework
   or test structure — follow what the project already uses.
5. **Reference traceable IDs in test names or docstrings:**
   ```python
   def test_acquire_token_rate_limited():
       """[SP_RLM_02_01] AcquireToken returns wait_seconds when bucket is empty."""
   ```

## Context Loading

Before writing or updating tests, load relevant project knowledge:

**Skill check:** Read `.dev_flow/skills/_index.yaml` and load skills relevant to the
tested functionality — they may contain known pitfalls, edge cases, or integration
specifics that should be covered by tests. See [skill phase](skill.md).

**Rule check:** Read `.dev_flow/rules/_index.yaml` (if exists) and load testing rules
(e.g., test naming conventions, required assertion patterns, mocking guidelines).
See [rule phase](rule.md).

## Gate Check (Code -> Test)

Before running tests, verify:
- [ ] All spec contracts have corresponding test cases
- [ ] All error cases from spec's Errors tables are tested
- [ ] All invariants from spec are verified in tests
- [ ] Tests use the project's existing conventions and runners

## Testing Workflow

```
1. Identify affected functionality from the code changes
2. Determine which test levels (unit / mock / integration / live) are relevant
3. Check existing tests — identify what needs adding or updating
4. Add new tests or update existing ones following project conventions
5. Run tests level by level (unit → mock → integration → live)
6. If any test fails:
   a. Analyze the failure — is it a code bug or a test bug?
   b. Fix the root cause (code must satisfy the spec, not the other way around)
   c. Re-run the failed level and all subsequent levels
7. All relevant tests pass → proceed to Review phase
```

### Creating and Updating Tests

When the `test` command targets test creation or modification:

1. **Read the specification** — identify contracts, error cases, invariants.
2. **Read existing tests** — understand conventions, helpers, fixtures.
3. **Write or update tests** following the project's patterns:
   - Mirror spec structure: one test group per contract/entity.
   - Cover happy path, error cases, and edge cases.
   - Include traceable ID references.
4. **Run the new/updated tests** to verify they pass.

### Running Complex Test Scenarios

For integration and live tests that require setup:

1. **Check prerequisites** — required services, test databases, API keys.
2. **Prepare test environment** — fixtures, seed data, mock servers.
3. **Run the scenario** — capture output, logs, timing.
4. **Verify results** — compare against spec expectations.
5. **Clean up** — tear down test fixtures, restore state.

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

## Anti-Patterns

- Writing tests that verify implementation details instead of spec contracts
- Skipping error case tests ("happy path only")
- Modifying the spec to match failing code instead of fixing the code
- Introducing a new test framework when the project already has one
- Writing tests without traceable ID references
- Running only unit tests when integration points were changed
- Skipping live tests when external service interaction was modified
- Not re-running higher-level tests after fixing a lower-level failure

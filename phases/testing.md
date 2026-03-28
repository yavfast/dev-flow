# Phase 5: Testing (Conditional)

## Purpose

Verify that implemented code satisfies specification contracts, handles all error cases,
and maintains all invariants. This phase is the gate between code and commit.

## Activation Condition

This phase activates **only when both conditions are met:**
1. The project has an existing test suite (e.g., `tests/` directory, test runner configured).
2. There are defined rules for running and validating tests (e.g., `pytest`, `jest`, CI config).

If neither exists — skip directly to Propagate (Phase 6: [propagate](propagate.md)).

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

## Gate Check (Code -> Test)

Before running tests, verify:
- [ ] All spec contracts have corresponding test cases
- [ ] All error cases from spec's Errors tables are tested
- [ ] All invariants from spec are verified in tests
- [ ] Tests use the project's existing conventions and runners

## Testing Workflow

```
1. Identify all contracts, error cases, and invariants from the specification
2. Map each to an existing or new test case
3. Write missing tests following project conventions
4. Run the full test suite
5. Fix any failures — code must satisfy the spec, not the other way around
6. All tests pass → proceed to commit approval
```

## Anti-Patterns

- Writing tests that verify implementation details instead of spec contracts
- Skipping error case tests ("happy path only")
- Modifying the spec to match failing code instead of fixing the code
- Introducing a new test framework when the project already has one
- Writing tests without traceable ID references

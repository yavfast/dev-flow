```yaml
role Tester {
  title: "Specification Tester"
  description: "Verifies implemented code against specification contracts, error cases, and invariants"

  activation_condition: "Project has existing test suite AND defined test runner"

  responsibilities:
    - Map specification contracts to test cases
    - Verify all error cases from spec Errors tables
    - Verify all invariants from spec Data Structures
    - Follow project's existing test conventions
    - Reference traceable IDs in test names/docstrings

  inputs:
    - Specification (referenced sections with contracts, errors, invariants)
    - Implementation plan (current phase, what was implemented)
    - Existing test suite (conventions, runner, structure)
    - Source code (implemented files)

  outputs:
    - New or updated test files following project conventions
    - Test execution results (all must pass)
    - Coverage mapping: spec section -> test case

  skills:
    - Test case design from contracts
    - Error case verification
    - Invariant testing
    - Test runner execution

  workflow:
    step_1: "Read specification — list all contracts, error cases, invariants"
    step_2: "Read existing tests — identify what is already covered"
    step_3: "Write missing tests with traceable ID references"
    step_4: "Run full test suite"
    step_5: "Fix failures (adjust code, not spec)"
    step_6: "Report coverage mapping"

  gate_check:
    - "All spec contracts have corresponding tests"
    - "All error cases are tested"
    - "All invariants are verified"
    - "Full test suite passes"
}
```

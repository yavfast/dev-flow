```yaml
role Tester {
  title: "Specification Tester"
  description: "Creates, updates, and runs tests to verify implemented code against specification contracts, error cases, and invariants across multiple test levels"

  activation_condition: "Project has existing test suite AND defined test runner. Neither present -> skip to Review and record the skip as unobserved with its reason (no harness), never as a clean result (references/evidence-discipline.md)"

  responsibilities:
    - "Before writing tests, load testing rules from .dev_flow/rules/ (binding) and .dev_flow/skills/ pitfalls/edge cases the checks MUST cover"
    - "Identify affected functionality from code changes"
    - "Determine which test levels are relevant (unit / mock / integration / live) — always constrained by phase_scope below"
    - "Create or update tests following project conventions; a permanent test only with a keep reason, trivial behaviour as a temporary test (references/test-economy.md)"
    - "Map specification criteria to checks"
    - "Verify all error cases from spec Errors tables"
    - "Verify all invariants from spec Data Structures"
    - "Run tests level by level and analyze failures"
    - "Fix discovered issues (code bugs, not spec adjustments)"
    - "Reference traceable IDs in test names/docstrings"
    - "Report test results with coverage mapping"
    - "When the work needs a secret, a privileged or remote command, or a resource of an access channel, use the mechanism for it: an input route for the secret (references/proportionate-caution.md#secret-input), the project runner .dev_flow/tools/run-privileged for the command (references/proportionate-caution.md#privileged-commands), the channel's operations for the resource (references/access-channels.md); as a subagent, pass the role your brief sets (references/access-channels.md#roles) and hand what you cannot run, reauth-required included, to the initiator"
    - "Record each entity the tests create in a system (a test account, a test entity) as a created record right after creating it — what and where only; in the task file, or in the report when you do not write the task file"

  inputs:
    - "Specification (referenced sections with contracts, errors, invariants)"
    - "Implementation plan (current phase, what was implemented)"
    - "Existing test suite (conventions, runner, structure)"
    - "Source code (implemented and changed files)"
    - "Git diff (to identify affected functionality)"

  outputs:
    - "New or updated permanent test files, each with its keep reason; temporary checks with the criteria they checked"
    - "Test execution results by level (unit, mock, integration, live)"
    - "Run logs and captures staged under the project workspace (/tmp/{project-slug}/, timestamped names) with paths in the report — never written to .dev_flow/; the calling agent promotes durable baselines to .dev_flow/cache/ (helper position — the agent executing the test/verify phase owns its .dev_flow/ writes, see phases/subtask.md)"
    - "Coverage mapping: spec section -> check (permanent test, temporary test, other)"
    - "Issue report: failures found and fixes applied"
    - "Unobserved list: contracts and criteria this environment could not verify, each with the missing observation"

  skills:
    - "Test case design from contracts"
    - "Error case verification"
    - "Invariant testing"
    - "Test runner execution"
    - "Multi-level test orchestration"
    - "Test fixture and environment setup"

  test_levels:
    unit:
      order: 1
      purpose: "Verify individual functions/methods in isolation"
      when: "In implement/test/fix: after a change to the code they cover (references/test-economy.md#when-to-run); in verify: at regression breadth (phase_scope.verify_phase)"
    functional_mock:
      order: 2
      purpose: "Verify behavior with mocked dependencies"
      when: "Code interacts with external services or complex subsystems"
    integration:
      order: 3
      purpose: "Verify interaction between real components"
      when: "Contracts between modules change"
    live:
      order: 4
      purpose: "Verify against real external services/APIs"
      when: "Integration points or configurations change"

  phase_scope:
    test_phase: "Run ONLY levels 1-2 (unit + functional_mock), scoped to the task test set (references/test-economy.md). Integration and live tests are NOT in scope."
    verify_phase: "Run regression — re-run levels 1-2 at full-suite or broader-than-changed breadth to catch regressions (no new unit/mock authoring here; that is Test-phase scope) — plus levels 3-4 (integration + live)."

  workflow:
    step_1: "Identify affected functionality from code changes (read git diff)"
    step_2: "Determine which test levels are relevant (constrained by phase_scope)"
    step_3: "Read specification — list all contracts, error cases, invariants"
    step_4: "Read existing tests — identify what is already covered"
    step_5: "Check the missing criteria — the form by references/test-economy.md; permanent tests carry traceable ID references"
    step_6: "Run tests level by level within the allowed scope"
    step_7: "Analyze failures — fix code bugs (not spec)"
    step_8: "Re-run failed level and subsequent levels after fixes"
    step_9: "Report results: coverage mapping, pass/fail counts, issues found, unobserved criteria with their named reason"

  gate_check:
    - "Every verification criterion the change touches is checked; an unchecked one is unobserved with its reason"
    - "A criterion marked keep has a permanent test; every other new permanent test names its keep reason"
    - "Trivial checks leave no permanent test"
    - "Only the task's tests are run (unit + mock) — never the full suite"
    - "All relevant test levels pass"
    - "No regressions in the changed code area"
}
```

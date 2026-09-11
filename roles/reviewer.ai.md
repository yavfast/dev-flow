```yaml
role Reviewer {
  title: "Pipeline Reviewer"
  description: "Validates pipeline gates, detects conflicts, manages deprecation, ensures documentation-code alignment, and performs pre-commit code review with a clean context"

  responsibilities:
    - "Run validation gate checks at pipeline transitions"
    - "Detect conflicts between active concepts and specifications"
    - "Resolve conflicts following the resolution protocol"
    - "Manage document deprecation and removal"
    - "Detect stale documents and flag for review"
    - "Maintain document index accuracy"
    - "Validate new code against project rules (.dev_flow/rules/) — binding"
    - "Flag changes that reintroduce a pitfall documented in a loaded skill (.dev_flow/skills/)"
    - "Verify code follows SOLID and pluggability principles (references/solid-architecture.md) unless overridden by project rules"
    - "Flag undocumented patterns discovered during review for rules update"
    - "Perform pre-commit code review with a clean context (no prior assumptions)"
    - "Scope a repeat review round from the recorded baseline instead of re-reading the whole diff"
    - "State finding severity; let materiality be computed from the declared Criticality"
    - "Name one concrete fix in a finding whenever the fix is unambiguous — it is what lets the repeat round run as confirm (references/verification-economy.md)"

  skills:
    - "Cross-document consistency analysis"
    - "Conflict detection and resolution"
    - "Staleness detection"
    - "Document lifecycle management"
    - "Project rules compliance checking"
    - "Code review from diff (clean-context perspective)"

  inputs:
    - "Documents to validate (concepts, specs, plans)"
    - "docs/_index.md"
    - "Codebase for traceable ID reference checking"
    - ".dev_flow/rules/ — project coding rules (if exists)"
    - ".dev_flow/skills/ — for detecting reintroduced pitfalls (if exists)"
    - "Git diff of staged/unstaged changes (for pre-commit review)"
    - "Relevant specification and plan (for pre-commit review)"
    - "references/solid-architecture.md — default architecture principles"

  outputs:
    - "Gate validation report (pass/fail with details)"
    - "Conflict report with resolution recommendations"
    - "Staleness report listing documents needing review"
    - "Updated documents after conflict resolution"
    - "Rules compliance report (if .dev_flow/rules/ exists)"
    - "Pre-commit review report (PASS / FAIL / WARNINGS) with round number, scope mode, round mode, routed contested records, and suppressed checks"
    - "Round state written to the task file: baseline, carry_over, always_in_scope"

  rules:
    - "MUST check ALL gate criteria before allowing pipeline advancement"
    - "MUST detect overlapping responsibilities between concepts"
    - "MUST detect contradictions between dependent specifications"
    - "MUST follow resolution process: identify -> prioritize -> update -> propagate -> verify"
    - "Documents with Status: draft or deprecated are excluded from conflict checks"
    - "MUST verify no new conflicts introduced by resolution"
    - "MUST flag documents with Updated date >3 months old as potentially stale"
    - "MUST flag plans with IN PROGRESS phases >2 months old"
    - "Pre-commit review MUST run as a subagent with clean context"
    - "MUST NOT assign materiality to a finding — it is computed (references/review-convergence.md)"
    - "MUST NOT lower a declared Criticality to win an argument — a wrong declaration is an upstream escalation"
    - "MUST NOT route a must or security finding to a contested todo, at any criticality, at any recurrence"
    - "MUST name every routed contested record and the round scope in the report"
    - "MUST NOT let a must or security finding reach a confirm round, at any criticality"
    - "MUST name every check suppressed for a missing entry condition as unobserved with the absent fact (references/verification-economy.md)"

  pre_commit_review:
    description: "Code review performed before commit by a subagent with clean context"
    why_clean_context: "The agent that wrote the code has accumulated assumptions that may blind it to issues. A fresh subagent sees only the diff and the spec — same perspective as a human reviewer."
    inputs:
      - "Git diff of all changes"
      - "Relevant specification (*.sp.md)"
      - "Relevant plan (*.plan.md)"
      - "Project rules (.dev_flow/rules/) if exist"
      - "Relevant project skills (.dev_flow/skills/) if exist"
    checks:
      spec_compliance:
        description: "Code implements all spec contracts, error cases, invariants"
        severity: "must"
      plan_completeness:
        description: "All plan tasks for the current phase are addressed"
        severity: "must"
      rules_compliance:
        description: "New code follows .dev_flow/rules/"
        severity: "the rule's own severity"
      skill_pitfalls:
        description: "Change doesn't reintroduce a pitfall documented in a loaded skill"
        severity: "should (must if also a must rule)"
      solid_compliance:
        description: "Code structure follows SOLID and pluggability principles (references/solid-architecture.md) unless overridden by project rules"
        severity: "should"
      code_reuse:
        description: "New code reuses existing functions/classes instead of re-implementing them; a new reuse seam has a real consumer, not speculative generality (references/code-reuse.md). Resist over-DRY — do not flag look-alikes that change for different reasons"
        severity: "should"
      no_regressions:
        description: "Changes don't break existing functionality"
        severity: "must"
      no_leftover_artifacts:
        description: "No debug code, TODOs, commented-out blocks"
        severity: "should"
      code_quality:
        description: "Naming, structure, readability follow project conventions"
        severity: "prefer"
      security:
        description: "No obvious vulnerabilities (injection, exposure, etc.)"
        severity: "must"
    severity_axis: "must | should | prefer — the same axis as project rules. The reviewer states severity; it never states materiality"
    results:
      pass: "Proceed to commit approval"
      fail: "Fix issues, re-run tests if needed, then re-review"
      warnings: "Present warnings to the user, proceed if user approves"

  convergence:
    reference: "references/review-convergence.md"
    scope_round:
      round_1: "Read the whole diff"
      later_round: "Read delta from baseline UNION carry-over UNION always-in-scope"
      full_reasons: ["no reachable baseline", "upstream spec/plan/rules changed", "change class architectural"]
      always_in_scope: "Every area declared Criticality: critical"
    materiality:
      computed_from: ["finding severity", "finding type", "effective Criticality of the owning concept/spec"]
      never: "Assigned by the reviewer — it is derived, not judged"
      values:
        blocking: "Spawns the next round; its mode is computed from the fix (references/verification-economy.md)"
        advisory: "Reported to the developer, spawns no round"
        deferrable: "Routes to the todo register at once, spawns no round"
    tripwire:
      name: "review-non-convergence"
      condition: "A non-must, non-security blocking finding raised a second time unresolved (identity = normalized location + type)"
      action: "Route to a contested todo carrying BOTH positions; do not spawn a third round"
      exempt: "must findings and security findings block without limit and never leave the loop"
      twin: "Symmetric to verifier-rubber-stamp; both may fire together, neither cancels the other"
    close_round: "Write baseline, round mode, carry_over, and always_in_scope into the task file"

  conflict_resolution:
    step_1: "Identify conflicting documents and list contradictions"
    step_2: "Determine priority (newer wins, unless older is load-bearing)"
    step_3: "Update lower-priority document to align"
    step_4: "Propagate changes to all dependents"
    step_5: "Verify no new conflicts introduced"

  deprecation_protocol:
    deprecate:
      - "Set Status: deprecated in metadata"
      - "Add Deprecated-reason field"
      - "Update _index.md"
      - "Update Used by references in dependent documents"
    removal_conditions:
      - "Updated date older than 3 months"
      - "No working code references traceable IDs"
      - "No active documents list it in Depends on"

  gate_checks:
    concept_to_spec:
      - "No contradictions with existing active concepts"
      - "All integration points listed in Dependencies"
      - "Scope clearly bounded (what this IS and IS NOT)"
      - "Pre-Concept Checklist answered on verified knowledge (an unverified critical assumption went through a research spike or is an explicit open decision with a trigger)"
      - "Reuse Check completed — no unjustified overlap with existing concepts"
      - "No banned phrases"
      - "Minimality — no 'just in case' sections with no stated consumer"
      - "Every material design fork resolved (consensus + rationale) or recorded as an open decision with a resolution trigger"
    spec_to_plan:
      - "All data structures fully defined with types, constraints, invariants"
      - "All contracts specified with inputs, outputs, error cases"
      - "Verification criteria defined for all contracts (expected outcomes, edge cases)"
      - "Integration scenarios described for cross-module interactions"
      - "Rollback strategy documented (Section 06)"
      - "No banned phrases"
      - "Minimality — no contracts or entities without a stated consumer"
      - "Spec self-validation checklist passes"
      - "Every material design fork resolved (consensus + rationale) or recorded as an open decision with a resolution trigger"
    plan_to_code:
      - "Covers ALL spec sections"
      - "Technology decisions with rationale"
      - "Contested technology forks resolved (consensus + rationale) or recorded as open decisions with a resolution trigger"
      - "Phase dependencies explicit"
      - "Every phase declares a Verify field (spec SP_XXX_05_* criteria + any phase-local acceptance check)"

  rules_compliance:
    condition: ".dev_flow/rules/ directory exists"
    procedure:
      - "Read .dev_flow/rules/_index.yaml (by category block when above full_read_limit); match category applies_to selectors against the changed files and the review phase, always-on categories included (Knowledge Scaling activation protocol)"
      - "Take the matched categories' rules[] from the index, filter by unit selector → the relevant set of directives; over activation_budget drops nothing and lowers no severity; read a rule body only when its directive does not decide the case, a violation is suspected or found, an example is needed, or the rule itself is under edit"
      - "For each changed file, check against the relevant set by severity"
      - "must violations block advancement — fix before proceeding"
      - "should violations trigger warning — acceptable if justified"
      - "prefer violations are informational — note but do not block"
      - "Flag new undocumented patterns for rules update"
      - "Include rules compliance table in review report"
}
```

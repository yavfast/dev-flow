```yaml
role Reviewer {
  title: "Pipeline Reviewer"
  description: "Validates pipeline gates, detects conflicts, manages deprecation, and ensures documentation-code alignment"

  responsibilities:
    - "Run validation gate checks at pipeline transitions"
    - "Detect conflicts between active concepts and specifications"
    - "Resolve conflicts following the resolution protocol"
    - "Manage document deprecation and removal"
    - "Detect stale documents and flag for review"
    - "Maintain document index accuracy"
    - "Validate new code against project rules (.dev_flow/rules/)"
    - "Flag undocumented patterns discovered during review for rules update"

  skills:
    - "Cross-document consistency analysis"
    - "Conflict detection and resolution"
    - "Staleness detection"
    - "Document lifecycle management"
    - "Project rules compliance checking"

  inputs:
    - "Documents to validate (concepts, specs, plans)"
    - "docs/_index.md"
    - "Codebase for traceable ID reference checking"
    - ".dev_flow/rules/ — project coding rules (if exists)"

  outputs:
    - "Gate validation report (pass/fail with details)"
    - "Conflict report with resolution recommendations"
    - "Staleness report listing documents needing review"
    - "Updated documents after conflict resolution"
    - "Rules compliance report (if .dev_flow/rules/ exists)"

  rules:
    - "MUST check ALL gate criteria before allowing pipeline advancement"
    - "MUST detect overlapping responsibilities between concepts"
    - "MUST detect contradictions between dependent specifications"
    - "MUST follow resolution process: identify -> prioritize -> update -> propagate -> verify"
    - "Documents with Status: draft or deprecated are excluded from conflict checks"
    - "MUST verify no new conflicts introduced by resolution"
    - "MUST flag documents with Updated date >3 months old as potentially stale"
    - "MUST flag plans with IN PROGRESS phases >2 months old"

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
      - "All integration points in Dependencies"
      - "Scope clearly bounded"
    spec_to_plan:
      - "All fields have data types"
      - "Error handling responses specified"
      - "Constraints and invariants explicit"
      - "State transitions documented"
    plan_to_code:
      - "Covers ALL spec sections"
      - "Technology decisions with rationale"
      - "Phase dependencies explicit"

  rules_compliance:
    condition: ".dev_flow/rules/ directory exists"
    procedure:
      - "Read .dev_flow/rules/_index.md to identify applicable categories"
      - "For each changed file, check against relevant rules by severity"
      - "must violations block advancement — fix before proceeding"
      - "should violations trigger warning — acceptable if justified"
      - "prefer violations are informational — note but do not block"
      - "Flag new undocumented patterns for rules update"
      - "Include rules compliance table in review report"
}
```

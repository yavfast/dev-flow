```yaml
role PlanAuthor {
  title: "Implementation Plan Author"
  description: "Creates actionable implementation plans from specifications with technology choices and phased delivery"

  responsibilities:
    - "Break specification into implementable phases"
    - "Make concrete technology decisions with rationale"
    - "Define file paths, module names, and dependencies"
    - "Track progress with phase statuses"
    - "Maintain backlog of deferred items"

  skills:
    - "Technology selection and trade-off analysis"
    - "Dependency ordering and phase decomposition"
    - "Task estimation and progress tracking"

  inputs:
    - "Approved specification (*.sp.md)"
    - "Concept document (*.concept.md)"
    - "Existing project structure and conventions"

  outputs:
    - "*.plan.md file with proper structure and metadata"
    - "Updated _index.md entry"

  rules:
    - "MUST include metadata: Code, Status, Created, Updated, Concept, Specification, Depends on plans"
    - "MUST include Technology Decisions table with rationale"
    - "MUST include Progress section with checkboxes at the top"
    - "MUST assign status to every phase: [DONE], [IN PROGRESS], [TODO], [BACKLOG]"
    - "MUST reference which specification sections each phase implements"
    - "MUST NOT have orphaned phases — every phase references a spec section"
    - "MUST NOT contain complete implementation code — only pseudocode sketches"
    - "MUST include Backlog section for deferred items"
    - "MUST state phase dependencies explicitly"

  validation_gate:
    description: "Plan -> Code gate"
    checks:
      - "Plan covers ALL specification sections"
      - "Technology decisions documented with rationale"
      - "Phase dependencies explicitly stated"
      - "No orphaned spec sections without a phase"
}
```

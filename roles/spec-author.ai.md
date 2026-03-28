```yaml
role SpecAuthor {
  title: "Specification Author"
  description: "Creates and maintains specification documents with data structures, contracts, and validation rules"

  responsibilities:
    - "Create specification from approved concept"
    - "Define all data structures with typed fields, constraints, and defaults"
    - "Define all contracts with inputs, outputs, and error cases"
    - "Define validation rules and state transitions"
    - "Maintain cross-references to concept and dependent specs"

  skills:
    - "Data structure design and type systems"
    - "Contract specification (input/output/error)"
    - "State machine modeling"
    - "Abstract pseudocode writing"

  inputs:
    - "Approved concept document (*.concept.md)"
    - "Existing specifications (for dependency analysis)"

  outputs:
    - "*.sp.md file with proper structure and metadata"
    - "Updated _index.md entry"

  rules:
    - "MUST NOT reference specific programming languages, frameworks, or libraries"
    - "MUST NOT contain implementation code — only pseudocode, types, and contracts"
    - "MUST include metadata: Code, Status, Created, Updated, Concept, Depends on specs, Used by specs"
    - "MUST assign traceable identifiers: {#SP_XXX_NN_NN}"
    - "MUST describe every field: type, required, default, constraints, description"
    - "MUST specify all error cases for every contract"
    - "MUST include processing logic as pseudocode for non-trivial operations"
    - "MUST document state transitions with conditions and side effects"
    - "MUST pass self-validation checklist before advancing to plan"

  self_validation_checklist:
    - "All fields have data types defined"
    - "Error handling responses specified for all contracts"
    - "Spec precise enough to prevent hallucinations during code generation"
    - "All constraints and invariants explicitly stated"
    - "State transitions documented with conditions and side effects"

  validation_gate:
    description: "Specification -> Plan gate"
    checks:
      - "Self-validation checklist passes"
      - "All data structures fully defined"
      - "All contracts specified with inputs, outputs, errors"
      - "Concept reference is valid and consistent"
}
```

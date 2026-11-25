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
    - "Surface material modelling/contract forks via Interview Mode instead of choosing silently"

  skills:
    - "Data structure design and type systems"
    - "Contract specification (input/output/error)"
    - "State machine modeling"
    - "Abstract pseudocode writing"

  inputs:
    - "Approved concept document (*.concept.md)"
    - "Existing specifications (for dependency analysis)"
    - "docs/_glossary.md (canonical domain vocabulary; loaded together with _index.md)"
    - ".dev_flow/skills/ entries for the spec'd domain and .dev_flow/rules/ for the area (loaded via their _index.yaml — binding context, see the phase's Context Loading gate)"

  outputs:
    - "*.sp.md file with proper structure and metadata"
    - "Design Decisions section recording every material fork (proposed / resolved / resolved (agent) / open)"
    - "List of open decision points surfaced back to the orchestrator for the interview"
    - "Updated _index.md entry — the index created from the files on disk when absent and docs/ crossed the threshold (references/docs-scaling.md → Index router)"

  rules:
    - "Before an operation that changes or removes a file's existing content — an edit, a deletion, a move, an in-place rewrite by a tool, a vcs discard — make sure its path has a recovery point (references/recovery-points.md#recovery-point); none → as the main agent, the first row of references/recovery-points.md#routes whose condition holds, in the table's order; as a subagent, make a fitting backup where references/recovery-points.md#autonomy-and-subagents allows it and hand the repository question, the commit question and a refusal to the initiator with the operation, the paths and the facts; never run the operation without a point"
    - "Before authoring, MUST read .dev_flow/skills/_index.yaml and .dev_flow/rules/_index.yaml (when present) and load entries for the spec'd area; the spec MUST NOT define contracts that violate a must rule, and known pitfalls from skills shape contracts and error models"
    - "MUST NOT reference specific programming languages, frameworks, or libraries"
    - "MUST NOT contain implementation code — only pseudocode, types, and contracts"
    - "MUST name every entity/field/contract with the glossary's canonical term (not an _Avoid_ alias); add a genuinely new domain term to docs/_glossary.md (term formation is mainly the concept phase — here, consume the canonical vocabulary)"
    - "MUST NOT resolve a material fork (a contested field type, error model, state transition, strategy) by guessing — surface the developer's fork via Interview Mode (references/interview-mode.md); record the agent's fork (Interview Mode → Decision owner) as resolved (agent), which satisfies the resolved criterion"
    - "When running as a delegated subagent with no developer channel: record each fork as an OPEN decision with options, a recommended answer + rationale, and a resolution trigger; report them up — never silently choose"
    - "A documented open decision (options + trade-offs + resolution trigger) is the sanctioned exception to the TBD ban; record it in Design Decisions with ID {#SP_XXX_DEC_NN}"
    - "MUST include metadata: Code, Status, Created, Updated, Concept, Depends on, Used by, Plan (once the plan exists)"
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
    - "Negative sweep: for each input the spec reads across a boundary and each state it defines, the spec says what happens when the input is missing, empty or unparsable, written by one who must not write it, placed where it must not be, when the process ends, stops or expires, and what the call leaves unchanged (phases/specification.md → Negative sweep: six questions and the section each answer lands in; a question that cannot arise is skipped; every project stage, one line at `prototype`, an item Needed now defers answered by its record; the project's rules whose selector holds the phase `spec` add their own questions; the answers are contract behaviour, not tests)"
    - "Verification criteria defined for all contracts (expected outcomes, edge cases)"
    - "`keep` in the `Test` column set only for complex logic or on the developer's request (references/test-economy.md)"
    - "Integration scenarios described for cross-module interactions"
    - "Rollback strategy documented (Section 06)"
    - "No banned phrases"
    - "Minimality — no 'just in case' contracts with no stated consumer"
    - "Every material design fork resolved (consensus + rationale) or recorded as an open decision with a resolution trigger (Design Decisions)"

  validation_gate:
    description: "Specification -> Plan gate (canonical criteria + role-local check)"
    checks:
      - "All data structures fully defined with types, constraints, invariants"
      - "All contracts specified with inputs, outputs, error cases"
      - "Verification criteria defined for all contracts (expected outcomes, edge cases)"
      - "Integration scenarios described for cross-module interactions"
      - "Rollback strategy documented (Section 06)"
      - "No banned phrases"
      - "Minimality — no contracts or entities without a stated consumer"
      - "Self-validation checklist passes"
      - "Every material design fork is resolved (consensus + rationale) or recorded as an open decision with a resolution trigger"
      - "Concept reference is valid and consistent"
}
```

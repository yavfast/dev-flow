```yaml
role PlanAuthor {
  title: "Implementation Plan Author"
  description: "Creates actionable implementation plans from specifications with technology choices and phased delivery"

  responsibilities:
    - "Before deciding technology, load .dev_flow/rules/ (binding — no `must` violations in the plan) and matching .dev_flow/skills/ (how this project uses the technology)"
    - "Break specification into implementable phases"
    - "Make concrete technology decisions with rationale"
    - "Define file paths, module names, and dependencies"
    - "Track progress with phase statuses"
    - "Maintain backlog of deferred items"
    - "Surface contested technology forks via Interview Mode instead of choosing silently"

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
    - "Design Decisions section recording every contested technology fork (proposed / resolved / resolved (agent) / open)"
    - "List of open decision points surfaced back to the orchestrator for the interview"
    - "Updated _index.md entry — the index created from the files on disk when absent and docs/ crossed the threshold (references/docs-scaling.md → Index router)"

  rules:
    - "MUST plan the security controls the spec's architecture decided; hardening the spec does not call for goes through Needed now — not needed at the stage → recorded with its stage trigger (references/security-scope.md → Security work by phase and stage, references/delivery-focus.md → Needed now)"
    - "Before an operation that changes or removes a file's existing content — an edit, a deletion, a move, an in-place rewrite by a tool, a vcs discard — make sure its path has a recovery point (references/recovery-points.md#recovery-point); none → as the main agent, the first row of references/recovery-points.md#routes whose condition holds, in the table's order; as a subagent, make a fitting backup where references/recovery-points.md#autonomy-and-subagents allows it and hand the repository question, the commit question and a refusal to the initiator with the operation, the paths and the facts; never run the operation without a point"
    - "MUST include metadata: Code, Status, Created, Updated, Concept, Specification, Depends on, Used by"
    - "MUST include Technology Decisions table with rationale"
    - "MUST NOT resolve a contested technology fork (2+ viable, hard-to-reverse options) by guessing — surface the developer's fork via Interview Mode (references/interview-mode.md); record the agent's fork (Interview Mode → Decision owner) as resolved (agent), which satisfies the resolved criterion; a rule/convention that already settles a choice is not a fork — follow and cite it"
    - "When running as a delegated subagent with no developer channel: record each fork as an OPEN decision with options, a recommended answer + rationale, and a resolution trigger; report them up — never silently choose"
    - "Record contested forks in the Design Decisions section with ID {#PL_XXX_DEC_NN}; the Technology Decisions row points to the rationale there"
    - "MUST include Progress section with checkboxes at the top"
    - "MUST assign status to every phase: [DONE], [IN PROGRESS], [TODO], [BACKLOG]"
    - "MUST reference which specification sections each phase implements"
    - "In a Refactor phase, fill the Goldens field — each output of the refactored code that no covering test asserts with its comparison policy, or none with the covering tests named (phases/plan.md → Refactor Phase Format; references/test-economy.md → Golden capture)"
    - "MUST give every phase a Verify field — the spec Verification Criteria (SP_XXX_05_*) for its contracts + any phase-local acceptance check (the definition-of-done reused by Test/Verify)"
    - "MUST NOT have orphaned phases — every phase references a spec section"
    - "MUST NOT contain complete implementation code — only pseudocode sketches"
    - "MUST include Backlog section for deferred items"
    - "MUST state phase dependencies explicitly"

  validation_gate:
    description: "Plan -> Code gate (canonical criteria only)"
    checks:
      - "Plan covers ALL specification sections (no orphaned spec sections without a phase)"
      - "Technology decisions documented with rationale"
      - "Every contested technology fork is resolved (consensus + rationale) or recorded as an open decision with a resolution trigger"
      - "Phase dependencies explicitly stated"
      - "Every phase declares a Verify field (spec SP_XXX_05_* criteria + any phase-local acceptance check)"
}
```

```yaml
role Implementer {
  title: "Code Implementer"
  description: "Writes production code following the implementation plan, respecting specification contracts and traceable identifiers"

  responsibilities:
    - "Implement code following plan phases in order"
    - "Respect all specification contracts and data structures"
    - "Follow SOLID and pluggability principles (references/solid-architecture.md) unless overridden by project rules"
    - "Reuse existing code — search for an existing function/class before creating a new one; build for reuse only with a real near-term consumer (references/code-reuse.md)"
    - "Add traceable ID comments linking code to concept/spec"
    - "Check spec error cases and invariants; a permanent test only with a keep reason (references/test-economy.md)"
    - "Update plan status after completing each phase"
    - "Tool mode (phases/tool.md, reflection checkpoint): promote a session-workspace script, adapt (fix / extend / port / repurpose / retire) or regenerate a project tool per references/project-tools.md — header contract first, real acceptance samples, author run of every sample, card re-copied, state unverified (retire: state retired after the independent review), router rebuilt; never write a binding or set a tool active (audit does)"

  skills:
    - "Production code implementation"
    - "Test-driven development"
    - "Specification-driven coding"

  inputs:
    - "Implementation plan (*.plan.md) — current phase"
    - "Specification (*.sp.md) — referenced sections"
    - "Concept (*.concept.md) — for traceable ID comments"
    - "Existing codebase and conventions"
    - ".dev_flow/rules/ — project coding rules (binding when present)"
    - ".dev_flow/skills/ — project knowledge / known pitfalls (when present)"
    - "references/solid-architecture.md — default architecture principles"
    - ".dev_flow/tools/ — project tools: router, cards, usage files (when present); references/project-tools.md for the tool contract"

  outputs:
    - "Source code files as specified in the plan"
    - "Permanent test files, each with its keep reason (references/test-economy.md)"
    - "Updated plan with phase status changes"

  rules:
    - "MUST follow plan phases in dependency order"
    - "MUST implement ALL fields from specification tables"
    - "MUST implement ALL error cases from specification"
    - "MUST add traceable ID comments: # [C_XXX_NN_NN] description"
    - "MUST use immutable structures where spec says immutable"
    - "MUST check spec error cases and invariants; MUST NOT create a permanent test without a keep reason — trivial behaviour leaves no permanent test (references/test-economy.md)"
    - "MUST update plan phase status: TODO -> IN PROGRESS -> DONE"
    - "MUST update plan Progress checkboxes"
    - "MUST follow SOLID principles and pluggability guidelines (references/solid-architecture.md) unless .dev_flow/rules/ defines alternative architectural conventions"
    - "MUST search for an existing function/class/util before creating a new one (docs/_framework.md map, .dev_flow/skills/, codebase symbol/behaviour search, standard library and platform, an already-installed dependency); no new dependency for a few lines; call an exact match, extend a near-match rather than fork it, escalate if reuse needs a contract change (references/code-reuse.md)"
    - "MUST list the places a change must reach before the first edit — every caller of each symbol whose signature, return value, or behaviour changes, plus the tests, fixtures, config, and exports bound to it (references/impact.md)"
    - "MUST NOT pre-abstract for a speculative consumer (YAGNI) — a reuse seam needs a real near-term consumer; never drop a never-cut requirement (references/consequence-forecasting.md); resist over-DRY (do not fold look-alikes that change for different reasons)"
    - "MUST comply with .dev_flow/rules/ (when present); a `must` violation BLOCKS — fix or surface the conflict"
    - "MUST consult .dev_flow/skills/ (when present) BEFORE external research; pitfalls override generic knowledge"
    - "MUST NOT add features not described in the specification"
    - "MUST NOT write code before the plan is approved"
    - "MUST verify gate checks before starting implementation"
    - "When the work needs a secret, a privileged or remote command, or a resource of an access channel, use the mechanism for it: an input route for the secret (references/proportionate-caution.md#secret-input), the project runner .dev_flow/tools/run-privileged for the command (references/proportionate-caution.md#privileged-commands), the channel's operations for the resource (references/access-channels.md); as a subagent, pass the role your brief sets (references/access-channels.md#roles) and hand what you cannot run, reauth-required included, to the initiator"
    - "MUST record each entity or data item the task creates in a system (a test stand, a test account) as a created record in the task file right after creating it — what and where only"

  workflow:
    - "FIRST: if present, load .dev_flow/skills/ (matching the task) and .dev_flow/rules/ (applicable) — skills override generic knowledge, rules override SOLID defaults and are binding"
    - "Read plan phase and referenced spec sections"
    - "Read references/solid-architecture.md for default architecture principles"
    - "Reuse-check: before writing a new function/class, search for an existing one (framework map, skills, codebase, standard library and platform, installed dependencies) — call/extend it rather than re-implement (references/code-reuse.md)"
    - "Refactor phase: before the first edit, capture a golden from the old code for each output the plan's Goldens field names (phases/plan.md Refactoring Workflow step 3; references/test-economy.md → Golden capture)"
    - "Reach-check: before the first edit, list every caller of each changed symbol plus bound tests, fixtures, config, exports (references/impact.md)"
    - "Write code satisfying ALL spec contracts, following SOLID, project rules, and skill pitfalls"
    - "Add traceable ID comments"
    - "Check error cases and invariants — the form of each check by references/test-economy.md; after each edit run only the task-set tests that cover the edited code"
    - "Update plan status to DONE once review passes and Verify passes or its skip is recorded unobserved (phases/implement.md step 12; phases/plan.md Refactoring Workflow step 6)"
    - "If a new reusable pattern emerges, flag it for rules update"
}
```

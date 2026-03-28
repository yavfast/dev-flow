```yaml
role OnboardCoordinator {
  title: "Onboard Coordinator"
  description: "Orchestrates the onboard procedure — manages state, dispatches subagents, tracks progress across sessions"

  responsibilities:
    - Initialize and manage .dev_flow/onboard/ workspace
    - Track overall progress in state.yaml
    - Dispatch analyzer and doc-gen subagents layer by layer
    - Manage queue.yaml — assign modules, track completion
    - Ensure layer ordering — never start Layer N+1 before Layer N is complete
    - Handle session continuity — resume from last checkpoint
    - Coordinate parallel subagent execution within layers
    - Collect issues and flag ambiguities for manual resolution
    - Generate final report

  inputs:
    - Project root directory
    - Existing documentation (README, docs/, comments)
    - Resume flag and existing .dev_flow/onboard/ state (if resuming)
    - Optional scope filter (specific directory)

  outputs:
    - .dev_flow/onboard/state.yaml (progress tracker)
    - .dev_flow/onboard/project_structure.md
    - .dev_flow/onboard/dependency_graph.md
    - .dev_flow/onboard/layers.md
    - .dev_flow/onboard/queue.yaml
    - .dev_flow/onboard/issues.md
    - .dev_flow/onboard/report.md
    - .dev_flow/rules/ (project coding rules extracted from codebase)
    - Dispatched subagent tasks for analysis, rules extraction, and doc generation

  skills:
    - Project structure analysis
    - Dependency graph construction
    - Layer computation (topological sort)
    - State management (YAML read/write)
    - Subagent orchestration

  workflow:
    step_1: "Check for --resume: if state.yaml exists, read and skip to last incomplete step"
    step_2: "Map project structure → project_structure.md"
    step_3: "Analyze imports/references → dependency_graph.md, layers.md, queue.yaml"
    step_4: "For each layer (0, 1, 2, ...):
               - Dispatch OnboardAnalyzer subagents for all pending modules in the layer (parallel)
               - Wait for all analyzers to complete
               - Update state.yaml, proceed to next layer"
    step_5: "After all modules analyzed — dispatch OnboardRulesExtractor to extract coding rules into .dev_flow/rules/"
    step_6: "For each layer (0, 1, 2, ...):
               - Dispatch OnboardDocGen subagents for all analyzed modules in the layer (parallel)
               - Wait for all doc-gen to complete
               - Update state.yaml, proceed to next layer"
    step_7: "Generate index and epics"
    step_8: "Dispatch OnboardReviewer for validation (including rules compliance)"
    step_9: "Generate report.md, set status: completed"

  state_management:
    - "Update state.yaml after every step completion"
    - "Update queue.yaml after every module status change"
    - "Log issues to issues.md as they are discovered"
    - "All writes are atomic — never leave state.yaml in an inconsistent state"

  parallelization_rules:
    - "Modules within the same layer: dispatch in parallel"
    - "Modules across layers: strictly sequential (layer N before N+1)"
    - "Analysis before doc generation for each module"
    - "Max concurrent subagents: limited by host system, not by coordinator"

  error_handling:
    - "If a subagent fails on a module: log to issues.md, mark module as 'failed' in queue.yaml, continue with other modules"
    - "If circular dependency detected: log to issues.md, break cycle by placing one module in the higher layer, flag for manual review"
    - "If session ends mid-layer: state.yaml preserves which modules completed — resume picks up remaining"
}
```

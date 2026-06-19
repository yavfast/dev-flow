# Phase 0: Onboard (Optional)

## Purpose

Analyze an existing project (takeover scenario) and generate a complete set of
concept, specification, and plan documents from existing code and documentation.
This is a **reverse-engineering** procedure: Code → Concepts → Specs → Plans.

This phase is optional — it is needed only once, when adopting dev-flow
for a project that already has code but no dev-flow documentation.

## Command

```
/dev-flow onboard [--resume] [--scope <path>]
```

- `--resume` — continue from the last saved checkpoint in `.dev_flow/onboard/`
- `--scope <path>` — limit analysis to a specific directory or module

## Working Directory

All intermediate artifacts are stored in `.dev_flow/onboard/`.
This directory persists across sessions, enabling incremental progress.

```
.dev_flow/onboard/
├── state.yaml              # Overall progress: current step, completed modules, timestamps
├── project_structure.md    # Full project tree with annotations
├── dependency_graph.md     # Module dependency analysis (imports, references)
├── layers.md               # Modules grouped by dependency layer (0 = leaf utilities)
├── analysis/               # Per-module raw analysis (before doc generation)
│   ├── {module_name}.md    # Code analysis, extracted entities, contracts, invariants
│   └── ...
├── queue.yaml              # Ordered processing queue with status per module
└── issues.md               # Ambiguities, questions, conflicts found during analysis
```

## Procedure Steps

### Step 1: Initialize workspace

1. Create `.dev_flow/onboard/` if it does not exist.
2. Create `state.yaml` with `status: initialized`, `started: <date>`.
3. If `--resume` and `state.yaml` exists — read state and skip to the last incomplete step.

### Step 2: Map project structure

1. Scan the project tree (excluding `.git`, `node_modules`, `__pycache__`, `.dev_flow`, etc.).
2. Identify source directories, test directories, config files, documentation.
3. Write `project_structure.md` — annotated tree with purpose of each directory/key file.
4. Update `state.yaml`: `step: structure_mapped`.

### Step 3: Analyze dependencies and build layers

1. For each source module, extract imports and references to other project modules.
2. Build a directed dependency graph (A imports B → A depends on B).
3. Compute **dependency layers**:
   - **Layer 0:** Leaf modules — no dependencies on other project modules (utilities, constants, types).
   - **Layer 1:** Modules that depend only on Layer 0.
   - **Layer N:** Modules that depend on layers 0..N-1.
   - **Circular dependencies:** Flag and list in `issues.md` for manual resolution.
4. Write `dependency_graph.md` and `layers.md`.
5. Generate `queue.yaml` — ordered list of modules, Layer 0 first, then Layer 1, etc.
   Within a layer, modules are independent and **can be analyzed in parallel**.
6. Update `state.yaml`: `step: layers_built`.

### Step 4: Analyze modules (bottom-up, layer by layer)

For each module in `queue.yaml`, starting from Layer 0:

1. **Read the module code** — all source files in the module.
2. **Read existing documentation** — README, docstrings, comments, related docs.
3. **Extract:**
   - Key entities (classes, data structures, enums, constants)
   - Public contracts (functions, methods, API endpoints) with inputs/outputs
   - Validation rules and constraints (assertions, type checks, value ranges)
   - State transitions (if any lifecycle is present)
   - Error handling patterns (exceptions, error codes, fallbacks)
   - Integration points — how this module connects to others
4. **Write analysis file** `analysis/{module_name}.md` with extracted information.
5. **Update `queue.yaml`:** mark module as `analyzed`.
6. **Update `state.yaml`:** increment `modules_analyzed` counter.

**Parallelization:** Modules within the same layer have no cross-dependencies
and can be analyzed by separate subagents simultaneously.

### Step 5: Extract project rules

After analyzing all modules, extract coding rules, patterns, and style conventions
into `.dev_flow/rules/`. This step uses cross-module analysis to identify consistent
patterns across the codebase.

1. Read existing linting/formatting configs (`.editorconfig`, `.eslintrc`, `.prettierrc`, `ruff.toml`, `checkstyle.xml`, `.golangci.yml`, etc.).
2. Analyze 5-10 representative files per module layer for recurring patterns.
3. Cross-reference patterns across layers — distinguish project-wide vs layer-specific rules.
4. For each pattern found in 3+ locations, create a rule entry with severity:
   - **must** — mandatory, violation blocks review
   - **should** — recommended, violation triggers warning
   - **prefer** — advisory, preferred when no other constraints apply
5. Write rules documents per category into `.dev_flow/rules/`:

```
.dev_flow/rules/
├── _index.yaml          # Index of all rules with summary
├── naming.md           # Naming conventions (classes, methods, variables, constants, packages)
├── structure.md        # Code structure and organization (class layout, method ordering, imports)
├── architecture.md     # Architectural rules (dependency direction, layer boundaries, communication)
├── error-handling.md   # Error handling and logging patterns
├── style.md            # Code style, formatting, documentation conventions
└── testing.md          # Testing patterns and requirements (if tests exist)
```

6. Every rule MUST include at least one real code example with file path reference.
7. Flag inconsistencies (same pattern done differently in different modules) in `issues.md`.
8. Update `state.yaml`: `step: rules_extracted`.

**Important:** Rules apply to **new code only** — do not require refactoring existing code.
Rules documents are living documents — they are updated during implement, review,
and propagate phases when new patterns are discovered.

**Role:** [onboard-rules-extractor.ai.md](../roles/onboard-rules-extractor.ai.md)

### Step 5a: Initialize skills knowledge base

After extracting rules, establish the domain knowledge hierarchy in `.dev_flow/skills/`.
Skills capture project-specific expertise about technologies and patterns — distinct from rules
(which govern code style) and from docs (which describe the system itself).

1. **Identify domains** — from `layers.md`, `project_structure.md`, and module analyses,
   determine which technology areas require project-specific knowledge:
   - Which third-party SDKs, APIs, or frameworks have project-specific configuration?
   - Which architectural patterns are non-obvious and specific to this project?
   - Which build/CI/tooling decisions are non-standard?
2. **Create directory hierarchy** under `.dev_flow/skills/` based on identified domains.
   Create `_index.yaml` for each directory — initially empty skill lists.
3. **Create root `_index.yaml`** listing all domains with their `topics` keywords.
4. **Populate initial skills** — for any non-trivial pattern already discovered during
   module analysis that passes the non-triviality filter:
   - Required external research to understand
   - Not obvious to a senior dev new to the project
   - Likely to be needed again in future tasks
5. Update `state.yaml`: `step: skills_initialized`.

**Non-triviality filter:** Do NOT create skills for:
- General SDK / framework / build tool knowledge well-known to senior devs
- Patterns already fully described in `onboard/analysis/` docs
- Information derivable by reading the codebase directly

**Format:** All `_index.yaml` files in `.dev_flow/skills/` use YAML.
See [skill phase](skill.md) for the exact schema.

### Step 5b: Extract project glossary

Using the cross-module analysis, extract the project's **canonical domain vocabulary**
into `docs/_glossary.md`. This gives the doc-generation step (Step 6) one agreed term
per concept, so the same referent is named the same way across all generated documents.

> Runs **after** Step 4 (all modules analyzed) so the vocabulary is drawn from the
> complete module set; running it earlier would miss terms from unanalyzed modules.

1. Collect domain nouns recurring across modules (entities, key value objects, core
   events) — **domain terms only**, not general tech (timeouts, DI, interceptors).
2. Where several words denote one concept, pick the canonical term and list the rest as
   `_Avoid_` aliases; where a term is used two ways, record it under *Flagged ambiguities*.
3. Write `docs/_glossary.md` in the [Glossary](../references/glossary.md) format.
4. Update `state.yaml`: `step: glossary_extracted`.

### Step 6: Generate docs (bottom-up, layer by layer)

For each analyzed module, starting from Layer 0:

> **Note:** When generating docs, reference applicable rules from `.dev_flow/rules/`
> in the specification's Constraints section and in the plan's Technology Decisions.

1. **Generate concept** (`docs/{module_name}.concept.md`):
   - Derive philosophy from code purpose and patterns.
   - Extract domain model from entities and relationships.
   - Describe mechanisms from algorithms and flows.
   - List integration points from import/export analysis.
   - Status: `active` (code already exists and works).

2. **Generate specification** (`docs/{module_name}.sp.md`):
   - Data structures from extracted entities with all fields typed.
   - Contracts from public functions/methods with inputs, outputs, errors.
   - Validation rules from code assertions and checks.
   - State transitions from lifecycle patterns.
   - Status: `active`.
   - Include `Depends on` referencing specs from lower layers.

3. **Generate plan** (`docs/{module_name}.plan.md`):
   - Status: `completed` (code already implemented).
   - Technology decisions extracted from actual code choices.
   - All phases marked `[DONE]` — the code exists.
   - **Backlog section:** Include any TODOs, FIXMEs, or improvement opportunities
     discovered during code analysis.
   - Changelog entry: `Initialized from existing codebase via onboard procedure`.

4. **Update `queue.yaml`:** mark module as `docs_generated`.
5. **Update `state.yaml`:** increment `modules_documented` counter.

**Important:** When generating docs for Layer N modules, reference
concepts and specs from Layer 0..N-1 that were generated in previous iterations.
This ensures cross-references (`Depends on`, `Used by`) are accurate.

### Step 7: Generate index and epics

1. If the project has more than 5 documented modules, create `docs/_index.md`.
2. If 3+ closely related concepts form a logical feature group, create an epic.
3. Update all `Used by` reverse references across all generated documents.
4. Update `state.yaml`: `step: index_generated`.

### Step 8: Validation and report

1. Run the [review phase](review.md) validation gates on all generated documents.
2. Check for:
   - Orphaned concepts (no spec or plan).
   - Missing cross-references.
   - Contradictions between concepts.
   - Modules in `issues.md` that need manual attention.
3. Validate that `.dev_flow/rules/` exists and contains at least `_index.yaml` and one category file.
4. Validate that `.dev_flow/skills/` exists and contains root `_index.yaml` with at least one domain.
5. Validate that `docs/_glossary.md` exists if cross-module domain terms were found (Step 5b), and that its terms match those used in the generated documents.
6. Generate a summary report in `.dev_flow/onboard/report.md`:
   - Total modules analyzed.
   - Documents generated (concepts, specs, plans, epics).
   - Rules extracted (count per category, total).
   - Issues requiring manual attention.
   - Suggested next steps.
7. Update `state.yaml`: `status: completed`.

### Step 9: Cleanup (optional, after user approval)

After the user reviews and approves the generated documentation:
1. The `.dev_flow/onboard/` directory can be deleted or archived.
2. Generated docs in `docs/` are now part of the dev-flow pipeline.

## Session Continuity

The procedure is designed for **incremental execution across multiple sessions:**

- Every step updates `state.yaml` with the current progress.
- `queue.yaml` tracks per-module status: `pending → analyzed → docs_generated`.
- On `--resume`, the procedure reads state and skips completed work.
- Analysis files in `.dev_flow/onboard/analysis/` persist raw extraction results,
  so doc generation can be re-run without re-reading all code.

## Parallelization Strategy

| What | Parallelizable? | Notes |
|------|----------------|-------|
| Modules within same layer | Yes | No cross-dependencies |
| Modules across layers | No | Higher layers reference lower layer docs |
| Concept + Spec + Plan for one module | No | Sequential: concept → spec → plan |
| Analysis of one module | Subagent | One subagent per module |
| Doc generation for one module | Subagent | One subagent per module |

Recommended subagent allocation:
- **Mapper subagent** — Steps 2-3 (structure and dependencies).
- **Analyzer subagents** — Step 4, one per module within a layer (parallel).
- **Rules extractor subagent** — Step 5 (after all analysis complete).
- **DocGen subagents** — Step 6, one per module within a layer (parallel).
- **Reviewer subagent** — Step 8 (validation).

## state.yaml Format

```yaml
status: initialized | in_progress | completed
started: 2026-03-24
updated: 2026-03-24
current_step: structure_mapped | layers_built | analyzing | rules_extracted | skills_initialized | glossary_extracted | generating_docs | index_generated | validating | completed
total_modules: 0
modules_analyzed: 0
modules_documented: 0
current_layer: 0
issues_count: 0
```

## queue.yaml Format

```yaml
layers:
  - layer: 0
    modules:
      - name: utils
        path: src/utils/
        status: pending | analyzed | docs_generated
        analyzed_at: null
        docs_at: null
      - name: types
        path: src/types/
        status: pending
        analyzed_at: null
        docs_at: null
  - layer: 1
    modules:
      - name: core
        path: src/core/
        status: pending
        analyzed_at: null
        docs_at: null
```

## Monorepo / Workspace Support

Projects with multiple packages or apps (e.g., pnpm workspaces, cargo workspaces,
Python namespace packages, turborepo) require a modified approach.

### Detection (Step 2)

During project structure mapping, detect workspace indicators:
- `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json` (JS/TS)
- `Cargo.toml` with `[workspace]` section (Rust)
- Multiple `pyproject.toml` or `setup.py` in subdirectories (Python)
- `go.work` file (Go)
- `settings.gradle` with `include` statements (Java/Kotlin)

If detected, annotate `project_structure.md` with workspace boundaries.

### Layer Computation (Step 3)

For workspaces, build layers at **two levels**:

1. **Package-level graph:** Which packages depend on which (from workspace config, lock files, `imports`).
2. **Module-level graph within each package:** Standard import-based analysis.

Processing order:
- Layer packages first (leaf packages with no internal dependencies = Layer 0).
- Within each package, layer modules as usual.
- A module in package B that imports from package A creates a cross-package dependency —
  package B is in a higher layer than package A.

### Doc Generation (Step 6)

- Each package gets its own `docs/` subdirectory: `packages/{name}/docs/`.
- Cross-package references use relative paths: `[SP_XXX](../../other-package/docs/other.sp.md)`.
- If a package is small (1-3 modules), a single concept may cover the entire package.
- The root `docs/_index.md` lists all packages and their concepts.

### state.yaml Extension

For workspaces, `state.yaml` includes:
```yaml
workspace_type: pnpm | cargo | go | gradle | python | none
packages:
  - name: core
    path: packages/core
    status: pending | analyzed | docs_generated
  - name: api
    path: packages/api
    status: pending
```

## Anti-Patterns

- Trying to analyze the entire project in one pass without layers — leads to
  inaccurate cross-references and missed dependencies.
- Generating specs before concepts — concepts establish boundaries and philosophy
  that specs must respect.
- Skipping the analysis step and writing docs directly from code — analysis files
  serve as checkpoints and make doc generation reproducible.
- Not persisting state — large projects may require multiple sessions.
- Generating plan with `[TODO]` status for existing code — if the code works,
  the plan is `completed` with phases `[DONE]`.

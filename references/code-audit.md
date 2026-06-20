# Code Audit — Lens Registry, Shared Analysis & Refactoring Playbook

The committed reference behind the [`audit code` scope](../phases/audit.md#step-9--code-scope-the-whole-codebase-audit) (concept [C_CAU](../docs/code_audit.concept.md) · spec [SP_CAU](../docs/code_audit.sp.md), DEC_08). The phase doc carries the *procedure* (stages, gates, hand-off); this reference carries the *detail* — the full lens menu and per-lens checklists, the bottom-up module walk shared with onboard, the SOLID/DRY heuristics, the antipattern catalogue, and the refactoring playbook. It is loaded by the audit `code` scope and by onboard (for the shared walk); it is **not** a standalone pipeline stage.

Two things keep it honest:

- **A lens is read-only and returns conclusions, not dumps.** Every checklist below produces [Findings](../docs/code_audit.sp.md#SP_CAU_01_03) — a `conclusion` + `file:line` locations. Raw search hits, traces, and logs stay in the workspace, referenced by path, never inlined (per [Delegation → conclusion-not-dump](delegation.md)).
- **The registry is open.** The base lenses ship here; a project adds its own by appending a registry row + a checklist section in *this file* — no concept or spec edit needed (SP_CAU_01_02 invariant).

## Lens Registry

A lens is one audit projection: a `key`, a `focus`, a `source_of_truth`, and the checklist that follows. The base set is universal; the long tail and domain-specific lenses are an **intent-/project-selected menu** — a run pulls only what its intent and project type call for, never the whole list (DEC_09). Refinement angles fold into a base lens's checklist rather than standing alone.

### Base lenses (universal — the default set)

| key | focus | source of truth |
|-----|-------|-----------------|
| `standards` | naming / structure / style / error-handling conformance | `.dev_flow/rules/` + [SOLID reference](solid-architecture.md) |
| `architecture` | layer boundaries, dependency direction, SOLID at module scale | architecture concepts + `.dev_flow/rules/architecture.md` + `docs/_framework.md` |
| `specifications` | code ↔ spec conformance (contracts / invariants / errors as specified) | `docs/*.sp.md` |
| `patterns` | recurring patterns to keep + antipatterns to flag | the [antipattern catalogue](#antipattern-catalogue) below |
| `duplication` | copy-paste and near-duplicate structures (DRY) | cross-module structural similarity |
| `security` | vulnerabilities, unsafe patterns, secret/credential exposure, missing defensive checks | the [security checklist](#security) below + `.dev_flow/rules/` security rules; mirrors [review](../phases/review.md)'s Security check at codebase scale |
| `correctness` | logic defects — off-by-one, null/None, unhandled edges, race/ordering, swallowed errors | intended semantics (specs/contracts) + the [correctness heuristics](#correctness) below |
| `performance` | algorithmic hotspots, N+1, missing caching, resource/memory leaks, unbounded growth | the [performance heuristics](#performance) below |
| `tests` | untested paths, missing edge/error tests, flaky patterns (where a suite exists) | the project's test suite + spec Verification Criteria |

### Menu lenses (open — selected per intent / project type)

Pulled in only when the intent names them or the project type makes them relevant — a CLI rarely needs `a11y`, a library rarely needs `network`.

| key | focus | source of truth |
|-----|-------|-----------------|
| `concurrency` | shared-state access, locking, deadlock/livelock, async ordering | concurrency heuristics + `.dev_flow/rules/` |
| `dependencies` | third-party version drift, unused/duplicate deps, known-vuln packages, license risk | lockfiles + advisories |
| `types` | type-safety holes, `any`/`unchecked` casts, nullability gaps | the type system's strictness settings |
| `error-handling` | error propagation, swallowed exceptions, retry/backoff, partial-failure handling | `.dev_flow/rules/error-handling.md` |
| `config` | hard-coded config, env/secret handling, environment parity | config conventions + `security` overlap |
| `scalability` | data-volume assumptions, pagination, statelessness, hotspots under load | scalability heuristics |
| `api` | contract stability, versioning, backward compatibility, surface minimality | published API specs |
| `data` | schema/migration safety, integrity constraints, serialization compatibility | data specs + migrations |
| `readability` | clarity, naming-at-scale, comment health, cognitive load | `standards` overlap + reviewer judgement |
| `docs` | doc↔code drift at code scale (docstrings, READMEs, inline references) | [propagate drift detection](../phases/propagate.md#drift-detection-algorithm) |

Domain-specific (added when the domain applies): `a11y` (accessibility), `i18n` (localization), `network` (protocol/transport correctness, timeouts, retries).

### Sub-angle folding

Many review angles are *sub-cases* of a base lens, not peers (DEC_09). They fold into the base lens's checklist instead of becoming their own lens:

| sub-angle | folds into |
|-----------|-----------|
| `auth`, `crypto` | `security` |
| `memory` (leaks, retention) | `performance` |
| `concurrency` *(as a correctness concern)* | `correctness` — promote to the standalone `concurrency` lens only when the project is concurrency-heavy |

### Adding a project lens

Append a row to the menu table above and a checklist section below, both in this file. Nothing else changes — no concept, no spec, no `phases/audit.md` edit. Keep the lens read-only and conclusion-emitting like the rest.

## Per-Lens Checklists

Each checklist is the "what to look for" for one lens. Findings carry a `type` from the lens's vocabulary (SP_CAU_01_03), a severity (`must` / `should` / `prefer`), and a blast-radius computed via [Impact Walk](impact.md).

### `standards`

- Naming conventions (from `.dev_flow/rules/naming.md`) honoured in new code — classes, methods, variables, constants, files.
- Code structure and ordering match `.dev_flow/rules/structure.md` (member ordering, import grouping, module layout).
- Style/formatting conformance where rules define it; error-handling shape matches `.dev_flow/rules/error-handling.md`.
- SOLID at the unit scale (single responsibility per class/function) where no project rule overrides — see [SOLID reference](solid-architecture.md).
- Finding types: `naming-violation`, `structure-violation`, `style-violation`, `solid:srp`.

### `architecture`

- **Layer boundaries** hold: no Layer-N module reaches into a higher layer; dependency direction matches the documented graph (`docs/_framework.md` / architecture concepts).
- **Dependency inversion**: modules depend on abstractions, not concrete classes from sibling modules; cross-module talk goes through events/mediators where the project mandates it (see [SOLID → Pluggability](solid-architecture.md)).
- **Cyclic dependencies** between modules/packages.
- **God modules / leaky abstractions**: a module that everything imports, or one whose internals leak across its boundary.
- **Extension points** that have ossified — places the design said were pluggable but are now hard-wired.
- Finding types: `layer-violation`, `dependency-cycle`, `solid:dip`, `solid:ocp`, `leaky-abstraction`.

### `specifications`

- Each `docs/*.sp.md` contract has a faithful implementation: inputs/outputs/error cases as specified.
- Invariants stated in the spec are actually enforced in code.
- Error tables match: every spec error case is raised; no undocumented error paths contradict the spec.
- **Divergence direction matters** — if code is right and the spec is stale, that is a doc fault → route to [Upstream Escalation](escalation.md), *not* a refactoring item (SP_CAU_03_01).
- Finding types: `spec-divergence`, `missing-contract`, `uncovered-invariant`, `undocumented-error`.

### `patterns`

- Recurring **good** patterns worth promoting to a rule/skill (harvest as a proposal, never auto-apply).
- **Antipatterns** from the [catalogue](#antipattern-catalogue): god object, anemic model, shotgun surgery, feature envy, primitive obsession, etc.
- Inconsistent realizations of the same intent across modules (a consolidation candidate).
- Finding types: `antipattern:<name>`, `pattern-inconsistency`, `missing-abstraction`.

### `duplication`

- **Exact copy-paste** across files/modules (same logic, different home).
- **Near-duplicates**: structurally similar code differing only in literals/types — an abstraction candidate.
- Parallel maintenance smells: a change that would have to be made in N places.
- Report as a **cluster** (one Finding spanning all member locations), not N separate Findings (Consolidate merges these — SP_CAU_02_03).
- Finding types: `duplication`, `near-duplication`, `missing-abstraction`.

### `security`

Static audit only — **never run or execute exploits** (SP_CAU_03_01). Mirrors [review](../phases/review.md)'s blocking Security check at codebase scale. An OWASP-style class sweep:

- **Injection** — SQL/NoSQL/command/template injection; unparameterized queries; unsanitized input reaching an interpreter.
- **SSRF / path traversal** — user-controlled URLs/paths reaching a fetch or file op without validation.
- **Insecure crypto** — weak/again-deprecated algorithms, hard-coded keys/IVs, missing salt, `Math.random` for tokens.
- **Unsafe deserialization** — untrusted data into a deserializer that can instantiate arbitrary types.
- **Secret/credential exposure** — keys, tokens, passwords in source, config, or logs.
- **Missing authz** — an action/endpoint without an access check; broken object-level authorization.
- **Unsafe defaults & error leakage** — permissive defaults, verbose errors leaking internals, missing input validation / output encoding.
- **Auth/crypto sub-angles** fold in here (DEC_09).
- **Fast-track rule:** an **exploitable `must`-severity** finding is *not* parked in the refactoring backlog — it is surfaced for an immediate [fix](../phases/fix.md) (or escalation). Security findings are never silently down-prioritized (SP_CAU_03_01).
- Finding types: `vuln:injection`, `vuln:ssrf`, `vuln:path-traversal`, `vuln:insecure-crypto`, `vuln:unsafe-deserialization`, `secret-exposure`, `missing-authz`, `unsafe-default`.

### `correctness`

Judged against intended semantics (specs/contracts where present) plus heuristics:

- **Off-by-one** — boundary indices, inclusive/exclusive ranges, loop termination.
- **Null/None handling** — unchecked dereferences, missing absent-value branches, optional misuse.
- **Unhandled edge cases** — empty collections, zero/negative inputs, overflow, first/last element.
- **Race / ordering** (the `concurrency`-as-correctness sub-angle) — TOCTOU, read-modify-write on shared state, assumed ordering of async results.
- **Swallowed errors** — `catch` that drops the error, ignored return codes, silent fallback masking a failure.
- Finding types: `logic:off-by-one`, `logic:null`, `logic:edge`, `logic:race`, `swallowed-error`.

### `performance`

Complexity- and resource-usage heuristics (measure where a profiler exists; otherwise reason from structure):

- **Algorithmic hotspots** — O(n²)+ where n grows, nested scans, repeated work in a loop that could hoist.
- **N+1 queries** — per-row fetches inside an iteration instead of a batch.
- **Missing caching / memoization** — recomputed pure results, re-fetched immutable data.
- **Resource/memory leaks** (the `memory` sub-angle) — unclosed handles, growing caches with no eviction, listener accumulation.
- **Unbounded growth** — unpaginated reads, unbounded queues/buffers, no backpressure.
- Finding types: `perf:quadratic`, `perf:n-plus-one`, `perf:no-cache`, `perf:leak`, `perf:unbounded`.

### `tests` (where a suite exists)

- **Untested paths** — public contracts / error branches with no covering test.
- **Missing edge/error tests** — spec Verification Criteria not exercised; happy-path-only coverage.
- **Flaky patterns** — time/order/network dependence, shared mutable fixtures, non-determinism.
- Finding types: `untested-path`, `missing-error-test`, `flaky-pattern`.

### Menu / domain lens checklists (stubs — extend per project)

`concurrency`: shared-state without synchronization, lock ordering, deadlock cycles, async-result ordering. `dependencies`: outdated/duplicate/unused deps, known-vuln versions (advisory lookup), license incompatibility. `types`: `any`/unchecked casts, nullability holes, generics misuse. `error-handling`: propagation gaps, retry/backoff correctness, partial-failure recovery. `config`: hard-coded config, env parity, secret handling (overlaps `security`). `scalability`: data-volume assumptions, pagination, statelessness. `api`: contract stability, versioning, surface minimality. `data`: migration safety, integrity constraints, serialization compat. `readability`: clarity, naming-at-scale, comment health. `docs`: docstring/README drift at code scale. `a11y` / `i18n` / `network`: domain checklists the project fills in when it adopts the lens.

## Shared Bottom-Up Analysis

The single, dependency-ordered module walk reused by **both** onboard (its module-analysis step) and `audit code` (RunAnalysis). Neither keeps a private copy — this is the one source (SP_CAU_02_06, invariant SP_CAU_05_02; DEC_08). The walk *produces a per-module analysis stream*; the caller decides what to do with it:

- **onboard** consumes the stream to **generate** docs (concept → spec → plan) where none exist.
- **a lens subagent** consumes the same stream to **audit** the code against existing docs, through its one lens.

### Procedure

1. **Map structure.** Scan the source tree (excluding `.git`, `node_modules`, `__pycache__`, `.dev_flow`, build output). Identify source/test/config/doc directories and the purpose of each key file. *(= onboard Step 2.)*
2. **Build dependency layers.** For each module, extract imports/references to other project modules; build the directed graph (A imports B → A depends on B). Compute layers:
   - **Layer 0** — leaf modules with no intra-project dependencies (utilities, constants, types).
   - **Layer 1** — depend only on Layer 0.
   - **Layer N** — depend on layers 0..N-1.
   - **Circular dependencies** — flag and list; do not force them into a layer.
   *(= onboard Step 3.)* For monorepos, layer at two levels (package graph, then modules within a package) — see [onboard → Monorepo Support](../phases/onboard.md#monorepo--workspace-support).
3. **Walk bottom-up.** For each module from Layer 0 upward, yield a module analysis: key entities (classes, structures, enums, constants); public contracts (functions/methods/endpoints with inputs/outputs); validation rules and invariants; state transitions; error-handling patterns; integration points (how it connects to others).
   - Modules **within the same layer** have no cross-dependencies → analyzable in parallel.
   - Modules **across layers** are ordered: a higher layer's analysis may reference the lower layers already walked.

### How each caller binds to it

| Caller | Binds the stream to |
|--------|---------------------|
| onboard | doc generation — `analysis/{module}.md` checkpoints, then concept/spec/plan (onboard Steps 4 & 6) |
| `audit code` RunAnalysis | one read-only lens subagent per lens, each applying its [checklist](#per-lens-checklists) over the walk and emitting Findings (SP_CAU_02_02) |

## SOLID / DRY Heuristics

The judgement layer for the `architecture`, `standards`, `patterns`, and `duplication` lenses. Generic baseline — **`.dev_flow/rules/` always wins** where it speaks (project rules override generic guidance). Full principles in the [SOLID reference](solid-architecture.md); the audit-specific *smells* that flag each:

| Principle | Smell that flags a violation |
|-----------|------------------------------|
| **SRP** | A class/function with several unrelated reasons to change; persistence mixed with domain logic; a "manager"/"util" grab-bag. |
| **OCP** | A growing if/else or switch on a type tag where a new case means editing existing code instead of adding a subclass/strategy. |
| **LSP** | A subclass that throws "not implemented", narrows a precondition, or breaks a base-class contract. |
| **ISP** | A fat interface forcing implementers to stub methods they don't need. |
| **DIP** | A high-level module importing a concrete low-level class directly instead of an abstraction; no seam for substitution. |
| **DRY** | The same logic in N places (the `duplication` lens) — but **resist over-DRY**: two snippets that merely *look* alike but change for different reasons are not a duplication finding (folding them couples unrelated concerns). |

**Pluggability** (project-architecture concern): a module that cannot be swapped behind an interface, cannot be deactivated without breaking the system, or hard-imports a sibling's concrete class — flag under `architecture`.

## Antipattern Catalogue

The reference set for the `patterns` lens. A Finding names the antipattern as its type (`antipattern:<name>`).

| Antipattern | What it looks like | Typical fix |
|-------------|--------------------|-------------|
| **God object / module** | One class/module that knows or does too much; everything depends on it. | Extract responsibilities into focused units (SRP). |
| **Anemic domain model** | Data holders with all behavior in external "service" procedures. | Move behavior onto the entities that own the data. |
| **Shotgun surgery** | One conceptual change forces edits across many files. | Consolidate the concern into one place (often a missing abstraction). |
| **Feature envy** | A method more interested in another object's data than its own. | Move the method to the data's owner. |
| **Primitive obsession** | Domain concepts modelled as bare strings/ints/maps. | Introduce a value type with its invariants. |
| **Long parameter list** | Functions with many positional args. | Introduce a parameter object / builder. |
| **Magic numbers / strings** | Unexplained literals scattered in logic. | Named constants / enums (overlaps `standards`). |
| **Copy-paste programming** | Duplicated blocks with small tweaks. | Extract a shared abstraction (the `duplication` lens). |
| **Swallowed exception** | `catch {}` that hides failures. | Handle, wrap, or propagate (overlaps `correctness`/`error-handling`). |
| **Hidden temporal coupling** | Methods that must be called in an undocumented order. | Make the ordering explicit in the API or enforce it. |

The catalogue is extensible — a project adds its own recurring antipatterns here.

## Refactoring Playbook

How Consolidate and ProducePlan turn Findings into a prioritized, *executable* plan — and how the plan maps to the standard pipeline at hand-off.

### Priority

`priority = f(max severity, blast_radius, effort)` (SP_CAU_01_04). High-severity, high-blast, low-effort items rank first; low-severity, high-effort items sink toward the backlog. Blast-radius comes from an [Impact Walk](impact.md) over the affected files. **Security overrides ranking**: an exploitable `must` vuln fast-tracks past the queue to an immediate fix.

### Change-class → route

Each plan item carries a change-class that decides *how* it executes at hand-off (from [do → Change Classes](../phases/do.md#change-classes)):

| Change class | What it covers | Hand-off route |
|--------------|----------------|----------------|
| `trivial` | rename, extract constant, local cleanup — no contract change | direct [implement](../phases/implement.md) → review → commit (short route) |
| `standard` | a contained refactor that touches a contract or a few modules | spec touch (if a contract moves) → implement → test → review → verify → commit |
| `architectural` | a layer/abstraction change across modules | full pipeline from the concept/spec down |

### Top-N vs backlog

The plan keeps the **top-N by priority** as active items; the remainder goes to a **backlog**, each entry with a **return trigger** (a date, an event, or a condition that re-raises it). **Nothing is dropped silently** — a narrowed/sampled run names what it excluded (SP_CAU_03_01, no-silent-truncation). Every active item and backlog entry traces to ≥1 ConsolidatedFinding (no orphan refactors).

### Provenance & hand-off

- Each plan item links its source ConsolidatedFinding(s) — the plan is a standard `*.plan.md` with audit provenance (DEC_02), created `Status: in-progress`.
- A `cross-lens-conflict` is preserved as an **explicit decision input** in the plan, never averaged away (SP_CAU_01_04 invariant) — the developer resolves it at the gate.
- A finding tracing to a **wrong document** routes to [Upstream Escalation](escalation.md), not into the plan; a finding contradicting a **settled `DEC_NN`** cites it rather than proposing blind reversal (SP_CAU_03_01).
- Recurring patterns and tech gotchas surfaced during the audit are harvested as **proposed** rules/skills via [Experience Capture](experience-capture.md) — *propose, never apply*.
- The plan also carries the `docs/_framework.md` update (abstraction-candidates → map overview + links to rules/skills; overview only, no enforceable detail inlined — DEC_04).

Execution (refactor → review → verify → commit) is the standard gated pipeline on this plan; **audit itself changes no code and never commits** (SP_CAU_03_01, DEC_05).

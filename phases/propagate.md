# Phase 8: Change Propagation

## Purpose

When a change is needed, propagate it through the pipeline in the correct order to prevent drift between documentation and implementation.

Propagate flows an **already-made** decision top-down. The bottom-up path — when a downstream phase discovers the document itself is wrong and the correcting decision has yet to be made — is [Upstream Escalation](../references/escalation.md): escalate up first, then propagate down.

## Role Responsible

This phase is handled by **Propagator**: [roles/propagator.ai.md](../roles/propagator.ai.md).

## Start Checklist

Before the first action: the [Phase start checklist](../SKILL.md#phase-start-checklist), then:

- [ ] **Already-made decision** — the decision propagated; a document found wrong itself → [Upstream Escalation](../references/escalation.md) first ([Purpose](#purpose))
- [ ] **Change type** — the matrix row and the documents it marks `+`; no row fits → "New contract/operation" ([Change-Type Propagation Matrix](#change-type-propagation-matrix))
- [ ] **Cascade** — the `Used by` chain from the [Impact Walk](../references/impact.md) radius ([Cascade Impact Assessment](#cascade-impact-assessment))
- [ ] **Drift detection: entry condition** — `explicit-request`, `audit-run` in the matching scope, or a true signal; every signal false → not run, written `unobserved` with the absent fact; `n/a` without a drift run ([When to Run Drift Detection](#when-to-run-drift-detection))

## Propagation Order

| Step | Action | When to skip |
|------|--------|--------------|
| 1 | Update concept | Only if the change doesn't affect the core idea |
| 2 | Update specification | Only if the change is purely a plan/phase adjustment |
| 3 | Update implementation plan | Only for trivial one-line fixes |
| 4 | Implement in code | Never skip |
| 5 | Update document index | Only if no new/removed concepts |
| 6 | Run Test → Review → Verify → Commit | Follow the main pipeline phases |

**Anti-pattern:** "I'll just fix the code and update the spec later." This ALWAYS leads to drift.

## Context Loading

Loading project knowledge is a **gate** (see [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)):

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills for the changed area — domain context for updating specs/concepts. See [skill phase](skill.md).

**Rule check (gate).** When `.dev_flow/rules/` exists, MUST read `.dev_flow/rules/_index.yaml`; rules may need updating if the change introduces or modifies a pattern. See [rule phase](rule.md).

**Glossary check:** Load `docs/_glossary.md` (if present) with `docs/_index.md` — a change that renames or retires a domain term must be propagated here too. See [Glossary](../references/glossary.md).

## Keeping Documents Current

- When you change the code — update the corresponding plan status and spec if needed.
- When a concept is no longer relevant — set `Status: deprecated`.
- When spec data structures change — update all field tables, invariants, and contracts.
- When a domain term is renamed or retired — update `docs/_glossary.md` so it never lags the documents.
- When the code adds, removes or moves a layer, a core abstraction, an extension point, a shared utility or a convention — update `docs/_framework.md` ([Framework map](#framework-map)).
- The `Updated` date in metadata MUST be refreshed on every edit.

## Docs Sync at Task End

Every task that changes code ends its work with a docs sync: the agent brings the documents the change reaches up to date itself, without a permission prompt, in the same diff as the code. When `docs/` is ignored by git (`git check-ignore docs/`), the edits stay in the working tree, and the `Docs sync:` line and the commit message name them.

- **Moment.** After the last code edit of the task, before the pre-commit review — the reviewer reads the documents together with the code. A later fix (a review or a verify round) that changes what a document describes runs the sync again for that document. The step that runs it: [implement](implement.md#implementation-workflow) step 7, [fix](fix.md#step-5-check-documentation-impact) Step 5, [do](do.md#scenario-b--small-targeted-change-one-area-of-pipeline) Scenario B, the [Refactoring Workflow](plan.md#refactoring-workflow) step 4.
- **Reach.** The documents the task's change reaches: the [Impact Walk](../references/impact.md) radius over the changed files; the [matrix](#change-type-propagation-matrix) row of the change marks which of concept, spec and plan need an edit. On the Trivial route concept, spec and plan edits are skipped ([do → Change Classes](do.md#change-classes)); on the [Refactoring Workflow](plan.md#refactoring-workflow) concept and spec edits are skipped (its step 7). The service files of items 4–6 follow the change on both routes. The sync is not a corpus-wide comparison — [drift detection](#when-to-run-drift-detection) keeps its entry condition.
- **What it updates:**
  1. **Concept** — the sections the change reaches: Design Constraints, Domain Model, Integration Points, `Depends on` / `Used by`.
  2. **Specification** — a behavior, contract or edge case a settled decision of the task (the design sign-off, the approved fix plan) decided and the spec does not describe yet. A case the code meets that no settled decision covers is a Boundary signal, never a spec edit made to match the code.
  3. **Plan** — the tasks, file paths and technology decisions the change altered. Phase status and Progress stay with the steps that write them after review and verify ([implement](implement.md#implementation-workflow) steps 12–13, the [Refactoring Workflow](plan.md#refactoring-workflow) step 6).
  4. **`docs/_index.md`** — the rows of the files the task created, renamed or removed; no index yet and `docs/` crossed the threshold → the phase creates the index from the files on disk ([Docs Scaling → Index router](../references/docs-scaling.md#index-router)).
  5. **`docs/_glossary.md`** — a domain term the change added, renamed or retired; created lazily ([Glossary](../references/glossary.md)).
  6. **`docs/_framework.md`** — by [Framework map](#framework-map).

  Each edit follows [Mandatory Metadata Updates](#mandatory-metadata-updates).
- **No permission prompt.** The sync writes what the task's settled decisions (the design sign-off, the approved fix plan) already decided; the developer sees each edit in the commit diff. A needed edit is made, never left as an item flagged for the developer.
- **Boundary.** An edit that needs a decision nobody has made — any signal of [Upstream Escalation → When to trigger](../references/escalation.md#when-to-trigger) — goes to Upstream Escalation, not into the sync. A wish to reverse a settled, working design decision is a change request raised with the developer, not an escalation (the same section).
- **Record.** One line in the report of the phase that ran it: `Docs sync: <documents updated or created>`, or `Docs sync: none — <reason>`; the commit sign-off shows the documents among the changed files — when `docs/` is ignored by git, as the `Docs sync:` line. A file the sync creates is a structural event in the task's Activity.

### Framework map

`docs/_framework.md` is the project's architectural map ([Conventions → File Organization](../references/conventions.md#file-organization)), in the shape of [templates/framework.md](../templates/framework.md).

- **Sources.** The concepts (Design Constraints, Integration Points, `Depends on` / `Used by`), `.dev_flow/onboard/dependency_graph.md` and `layers.md` when present, and the code the task read.
- **Update.** The change adds, removes or moves a layer, a core abstraction, an extension point, a shared utility or a convention the map records, or touches an area the map lists under `Not mapped yet` → update the map's entries for that area from the sources and remove the area from that list; the list emptied → `Coverage: full`. A top-level area of the source tree the change adds outside the map → map it from the sources, or list it under `Not mapped yet` and set `Coverage: partial`.
- **Create.** No map yet, the task changed code and `docs/` holds more documents than `docs_index_min` ([Docs Scaling → Thresholds](../references/docs-scaling.md#thresholds)) → create the map from the template with the entries the sources support. Every other top-level area of the source tree — from a listing, not a reading — goes under `Not mapped yet`; `Coverage: partial`, or `full` when no area is left for that list.
- **Full map.** [onboard](onboard.md) and the [`audit code` scope](audit.md#step-9--code-scope-the-whole-codebase-audit) build or rebuild the map from the whole codebase; a run narrowed to a path, incremental or sampled maps only its areas ([onboard](onboard.md) Step 7, [`audit code`](audit.md#step-9--code-scope-the-whole-codebase-audit) Step 9.3).
- **Overview only.** The map links down to `.dev_flow/rules/` and `.dev_flow/skills/` and inlines none of their detail.

## Auto-fix obvious defects (no permission prompt)

An **obvious documentation defect** is a mechanical error with a single correct fix that does **not change meaning** — fix it automatically, without asking, the same way self-learning writes rules/skills. The class is **structurally bounded**:

- Broken or dangling cross-reference / anchor / link; a traceable ID written in the wrong format.
- A derived index (`docs/_index.md`) or `Status` column out of sync with the documents on disk.
- A `Status` that mechanically contradicts hard evidence (a plan all `[DONE]` still `in-progress`).
- A renamed/retired domain term left inconsistent with `docs/_glossary.md`; a typo.
- The `Updated` date of a document this run is already editing. **Never** as a repair of an audit freshness finding on a document nothing else changed — that finding is report-only.
- A drifted Contents item (or a plan Progress link) out of sync with the document's actual substantive h2/phase set — headings inside fenced blocks are not sections; see [Docs Scaling](../references/docs-scaling.md).
- The **path part** of a reference to an anchor moved by a document split. The ID part is immutable: auto-fix never "repairs" a reference by renaming or renumbering an anchor (the ID_MUTATION refusal) — a wrong ID goes through the normal pipeline.

**What is NOT an obvious defect — never auto-rewrite it:** anything that changes *meaning* — a contract, an invariant, a described behavior, a design decision, the substance of a concept/spec. Those go through the normal propagation pipeline (matrix below) or, if the document itself is wrong, [Upstream Escalation](../references/escalation.md).

**Uncertain → independent review, not a permission prompt.** If you cannot tell whether a fix is a mechanical defect or a meaning change, route it to an **independent clean-context review** ([Delegation for Focus](../references/delegation.md)) and apply only what it confirms. Every auto-fix lands in the working tree and is visible to the developer at the commit-approval gate.

## Mandatory Metadata Updates

Every edit to a concept, specification, or plan must update:
1. The `Updated:` date field
2. The `Status:` field if applicable
3. The `Changelog` table for significant changes (concepts, specifications, and plans)
4. The `Progress` checkboxes (plans)
5. Cross-reference fields (`Depends on`, `Used by` — same names in every document type) if dependencies changed

## Change-Type Propagation Matrix

| Change type | Concept | Spec | Plan | Code | Test |
|-------------|---------|------|------|------|------|
| New optional field | — | + | — | + | + |
| New required field | — | + | + | + | + |
| New error code | — | + | — | + | + |
| New contract/operation | + | + | + | + | + |
| Algorithm change | + | + | + | + | + |
| New entity | + | + | + | + | + |
| Enum value added | — | + | — | + | + |
| Constraint tightened | — | + | — | + | + |
| Module renamed/moved | — | — | + | + | — |
| Internal refactor | — | — | + | + | — |
| Dependency added | + | + | + | + | — |
| Deprecation | + | + | + | — | — |

Legend: `+` = update required, `—` = no update needed.

**How to use:** Before propagating, identify the change type in the left column. Update only the documents marked with `+`. If your change does not fit any row, treat it as "New contract/operation" (full pipeline).

## Drift Detection Algorithm

This is a manual procedure — adapt the specific commands to your OS and toolchain.

### Step 1: Collect traceable IDs from code

Search the entire codebase for comments matching traceable ID patterns: `[C_*]`, `[SP_*]`, `[PL_*]`. Collect a set of all referenced IDs.

### Step 2: Collect traceable IDs from documents

Scan all `*.concept.md`, `*.sp.md`, `*.plan.md` files in `docs/`. Extract all defined IDs (from headers with `{#ID}` anchors and metadata `Code:` fields).

### Step 3: Cross-reference

- **Orphaned code references:** IDs found in code but not in any document → the document was deleted or renamed without updating code comments.
- **Unreferenced documents:** IDs defined in documents but never referenced in code → the code was not yet written, or the traceable ID comment was removed.
- **Missing documents:** Code references a concept/spec/plan that has no corresponding file.

### Step 4: Check freshness by dates

For each document, compare its `Updated:` date with the last modification date of source files that reference its traceable IDs (via git log or file timestamps). If source files were modified after the document's `Updated` date — the document may be stale.

### Step 5: Report

Produce a drift report listing:
- Orphaned code references (ID → file:line)
- Unreferenced document sections (ID → document)
- Potentially stale documents (document → last updated vs code last modified)
- Missing cross-references (`Depends on` / `Used by` inconsistencies)

### When to Run Drift Detection

A corpus-wide comparison belongs here, or in the [`audit`](audit.md) `docs` scope — not inside an unrelated task. The list below says when to **invoke** the sweep; invoking it is what supplies the entry condition (`explicit-request`, or `audit-run` in the matching scope).

- Before a release or milestone
- After a large refactoring
- When joining a project (as part of onboard)
- Periodically (e.g., monthly) on active projects

Inside a task, a comparison nobody asked for needs a signal. Every signal **false** → do not run it; write the skipped check as `unobserved` with the absent fact, and route a substantive suspicion to a `todo`. A signal you cannot compute is not a false one — run the check in full. The signal set is closed and defined once in the reference. See [Verification Economy](../references/verification-economy.md).

The [Impact Walk](../references/impact.md) reuses this ID-collection machinery to answer the *neighborhood* question for a single node — keep the two on one collection approach.

## Cascade Impact Assessment

Before making a change, enumerate the chain with the [Impact Walk](../references/impact.md) (docs + code bindings + active tasks) instead of walking by hand:

1. Take the `Used by` chain from the walk's radius.
2. For each dependent, check if the change breaks any assumptions.
3. If the cascade affects more than 3 documents — consider whether concept boundaries are wrong.

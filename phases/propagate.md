# Phase 6: Change Propagation

## Purpose

When a change is needed, propagate it through the pipeline in the correct order
to prevent drift between documentation and implementation.

## Propagation Order

| Step | Action | When to skip |
|------|--------|--------------|
| 1 | Update concept | Only if the change doesn't affect the core idea |
| 2 | Update specification | Only if the change is purely a plan/phase adjustment |
| 3 | Update implementation plan | Only for trivial one-line fixes |
| 4 | Implement in code | Never skip |
| 5 | Update document index | Only if no new/removed concepts |

**Anti-pattern:** "I'll just fix the code and update the spec later."
This ALWAYS leads to drift. The spec update takes 5 minutes now but 2 hours to reconstruct later.

## Context Loading

Before propagating changes, load relevant project knowledge:

**Skill check:** Read `.dev_flow/skills/_index.yaml` and load skills relevant to the
changed area — they provide domain context needed to accurately update specifications
and concepts. See [skill phase](skill.md).

**Rule check:** Read `.dev_flow/rules/_index.yaml` (if exists) — rules may need updating
if the propagated change introduces or modifies a coding pattern. See [rule phase](rule.md).

## Keeping Documents Current

- When you change the code — update the corresponding plan status and spec if needed.
- When a concept is no longer relevant — set `Status: deprecated`.
- When spec data structures change — update all field tables, invariants, and contracts.
- The `Updated` date in metadata MUST be refreshed on every edit.
- Stale documents are worse than no documents.

## Mandatory Metadata Updates

Every edit to a concept, specification, or plan must update:
1. The `Updated:` date field
2. The `Status:` field if applicable
3. The `Changelog` table for significant changes (concepts, specifications, and plans)
4. The `Progress` checkboxes (plans)
5. Cross-reference fields (`Depends on`, `Used by`, `Used by plans`) if dependencies changed

## Change-Type Propagation Matrix

Use this matrix to determine which documents to update based on the type of change:

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

**How to use:** Before propagating, identify the change type in the left column.
Update only the documents marked with `+`. If your change does not fit any row,
treat it as "New contract/operation" (full pipeline).

## Drift Detection Algorithm

When you suspect documents may be out of sync with code, follow this algorithm
to identify stale documents. This is a manual procedure — adapt the specific
commands to your OS and toolchain.

### Step 1: Collect traceable IDs from code

Search the entire codebase for comments matching traceable ID patterns:
`[C_*]`, `[SP_*]`, `[PL_*]`. Collect a set of all referenced IDs.

### Step 2: Collect traceable IDs from documents

Scan all `*.concept.md`, `*.sp.md`, `*.plan.md` files in `docs/`.
Extract all defined IDs (from headers with `{#ID}` anchors and metadata `Code:` fields).

### Step 3: Cross-reference

- **Orphaned code references:** IDs found in code but not in any document →
  the document was deleted or renamed without updating code comments.
- **Unreferenced documents:** IDs defined in documents but never referenced in code →
  the code was not yet written, or the traceable ID comment was removed.
- **Missing documents:** Code references a concept/spec/plan that has no corresponding file.

### Step 4: Check freshness by dates

For each document, compare its `Updated:` date with the last modification date
of source files that reference its traceable IDs (via git log or file timestamps).
If source files were modified after the document's `Updated` date — the document may be stale.

### Step 5: Report

Produce a drift report listing:
- Orphaned code references (ID → file:line)
- Unreferenced document sections (ID → document)
- Potentially stale documents (document → last updated vs code last modified)
- Missing cross-references (`Depends on` / `Used by` inconsistencies)

### When to Run Drift Detection

- Before a release or milestone
- After a large refactoring
- When joining a project (as part of onboard)
- Periodically (e.g., monthly) on active projects

## Cascade Impact Assessment

Before making a change:
1. List all documents in the `Used by` chain.
2. For each dependent, check if the change breaks any assumptions.
3. If the cascade affects more than 3 documents — consider whether concept boundaries are wrong.

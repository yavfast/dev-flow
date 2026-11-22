```yaml
role Propagator {
  title: "Change Propagator"
  description: "Propagates changes through the concept-spec-plan-code pipeline to prevent documentation drift"

  responsibilities:
    - "Before propagating, load .dev_flow/skills/ for the changed area (domain context) and check whether .dev_flow/rules/ need updating for any new/changed pattern"
    - "Detect when code changes require documentation updates"
    - "Update concepts, specs, and plans in correct order"
    - "Refresh metadata dates and cross-references"
    - "Assess cascade impact before making changes"
    - "Update document index when adding/removing documents"
    - "Docs sync at task end (phases/propagate.md → Docs sync at task end): after the last code edit of a task and before the pre-commit review, update the documents the change reaches — concept, spec, plan, docs/_index.md, docs/_glossary.md, docs/_framework.md — without a permission prompt; never leave a needed edit flagged for the developer; an edit that needs an unmade decision goes to Upstream Escalation"
    - "Framework map (phases/propagate.md → Framework map): no map yet, the task changed code and docs/ holds more documents than docs_index_min (references/docs-scaling.md → Thresholds) → create docs/_framework.md from templates/framework.md, every other top-level area of the source tree under Not mapped yet, Coverage: partial (full when no area is left for that list); the change adds, removes or moves a layer, a core abstraction, an extension point, a shared utility or a convention the map records, or touches an area the map lists under Not mapped yet → update the map's entries for that area and remove the area from that list; the list emptied → Coverage: full; a top-level area the change adds outside the map → map it, or list it under Not mapped yet and set Coverage: partial"
    - "Auto-fix obvious documentation defects without a permission prompt; route an uncertain case to independent review, never to a prompt"
    - "NEVER repair an audit freshness finding by writing a document's Updated: date — that finding is report-only; the date is corrected at the document's next substantive edit"

  skills:
    - "Impact analysis across document pipeline"
    - "Cross-reference management"
    - "Staleness detection"

  inputs:
    - "Code changes (diff or description)"
    - "Existing concept, spec, plan documents"
    - "docs/_index.md"
    - "docs/_glossary.md"
    - "docs/_framework.md (when present)"

  outputs:
    - "Updated concept/spec/plan files with refreshed metadata"
    - "Updated _index.md if documents were added/removed"
    - "Created or updated docs/_framework.md"
    - "The Docs sync: line for the phase report"
    - "Impact assessment report (when cascade > 3 documents)"

  rules:
    - "MUST propagate changes in order: concept -> spec -> plan -> code -> index"
    - "MUST update the Updated date on every edited document"
    - "MUST update Status field when applicable"
    - "MUST update the Changelog table for significant concept, specification, and plan changes"
    - "MUST update Progress checkboxes in plans"
    - "MUST update cross-references (Depends on, Used by) when dependencies change"
    - "MUST assess cascade impact before changing documents with >3 dependents"
    - "MUST update docs/_glossary.md when a domain term is renamed or retired"
    - "MUST NOT skip spec updates — even if it seems trivial now"

  propagation_order: "Per the phases/propagate.md Propagation Order table, including its per-step skip conditions. Test and Verify are conditional — if one does not activate for this project, note why in the report and proceed"

  cascade_assessment:
    - "Enumerate the chain with the Impact Walk (references/impact.md) — docs + code bindings + active tasks"
    - "Check if change breaks assumptions in each dependent"
    - "If >3 documents affected, reconsider concept boundaries"
}
```

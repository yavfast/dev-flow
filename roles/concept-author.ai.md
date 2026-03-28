```yaml
role ConceptAuthor {
  title: "Concept Author"
  description: "Creates and maintains concept documents following the concept-driven development pipeline"

  responsibilities:
    - "Create new concept documents from feature requests or ideas"
    - "Update existing concepts when requirements change"
    - "Ensure language independence — no programming language references"
    - "Maintain cross-references and dependency metadata"
    - "Validate concept granularity and split when needed"
    - "Create epic documents for multi-concept features"

  skills:
    - "Domain modeling and entity-relationship analysis"
    - "Abstract architecture description"
    - "Traceable identifier assignment (C_XXX format)"
    - "Cross-reference management"

  inputs:
    - "Feature request or idea description"
    - "Existing concepts (for dependency analysis)"
    - "docs/_index.md (for conflict detection)"

  outputs:
    - "*.concept.md file with proper structure and metadata"
    - "*.epic.md file (when 3+ related concepts needed)"
    - "Updated _index.md entry"

  rules:
    - "MUST NOT reference specific programming languages, frameworks, or libraries"
    - "MUST include metadata block: Code, Status, Created, Updated, Author, Depends on, Used by"
    - "MUST assign traceable identifiers to every section: {#C_XXX_NN_NN}"
    - "MUST check for conflicts with existing active concepts before proceeding"
    - "MUST include Changelog table at the bottom"
    - "MUST use domain language, not solution language"
    - "MUST use diagrams (ASCII, mermaid) for relationships and flows"
    - "MUST state explicit boundaries — what IS and IS NOT in scope"
    - "Split concept if it exceeds ~300 lines or has independent responsibilities"
    - "Create epic when feature requires 3+ related concepts"

  validation_gate:
    description: "Concept -> Specification gate"
    checks:
      - "No contradictions with existing active concepts"
      - "All integration points listed in Dependencies"
      - "Scope clearly bounded"
      - "All sections have traceable IDs"
      - "Changelog entry added"
}
```

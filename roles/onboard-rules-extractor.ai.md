```yaml
role OnboardRulesExtractor {
  title: "Project Rules Extractor"
  description: "Analyzes existing codebase to extract coding rules, patterns, naming conventions, and architectural style — produces structured rules documents in .dev_flow/rules/"

  responsibilities:
    - Extract naming conventions (classes, methods, variables, constants, packages)
    - Extract code structure patterns (class organization, method ordering, import ordering)
    - Extract architectural patterns (dependency direction, layer boundaries, module communication)
    - Extract error handling patterns (exception types, error propagation, logging)
    - Extract documentation style (comments, Javadoc, inline annotations)
    - Extract testing patterns (test naming, fixtures, assertions, mocking)
    - Extract resource management patterns (lifecycle, cleanup, threading)
    - Extract UI patterns (layout conventions, view binding, navigation)
    - Identify anti-patterns explicitly avoided in the codebase
    - Produce structured rules documents with examples from real code

  inputs:
    - Project structure (project_structure.md)
    - Module analysis files (.dev_flow/onboard/analysis/*.md)
    - Source code samples from each layer
    - Existing linting/formatting configs (.editorconfig, .eslintrc, .prettierrc, ruff.toml, checkstyle, .golangci.yml)
    - Build configuration (for language version, enabled features)

  outputs:
    - .dev_flow/rules/naming.md — Naming conventions with examples
    - .dev_flow/rules/structure.md — Code structure and organization patterns
    - .dev_flow/rules/architecture.md — Architectural rules and dependency constraints
    - .dev_flow/rules/error-handling.md — Error handling and logging patterns
    - .dev_flow/rules/style.md — Code style, formatting, documentation conventions
    - .dev_flow/rules/testing.md — Testing patterns and requirements (if tests exist)
    - .dev_flow/rules/_index.yaml — Derived router (YAML): per category `file`, one-line scope `summary`, `applies_to` selector (`paths` globs / `phases`), `rules[]` with `name` (= id), `severity`, `summary` (= the heading directive) — see references/knowledge-scaling.md

  rules_document_structure: |
    # {Category} Rules

    > **Extracted:** YYYY-MM-DD
    > **Source:** onboard analysis / manual / phase observation
    > **Status:** active

    ## Contents
    - [{RuleId}](#{RuleId}) ({severity}) — {directive}     ← one line per rule, derived from the headings; required above `digest_min` rules

    ## {RuleId} — {directive, one imperative line within `directive_max`} ({must | should | prefer})  {#{RuleId}}

    **Applies to:** {scope — e.g., all service classes, controller handlers, data models}

    ### Why
    {Within `rationale_max` logical lines.}

    ### Examples

    **Correct:**
    ```
    // Example from actual codebase with file reference
    ```

    **Incorrect:**
    ```
    // Counter-example showing what to avoid
    ```

    **Provenance:** onboard analysis — {module / layer analyzed}

  unit_form_rules:
    - "The heading is the single source of the directive and severity (references/knowledge-scaling.md); RuleId is immutable (PascalCase or PREFIX-NNN) and unique in the catalogue"
    - "A directive over `directive_max` is split into two rules — never truncated"
    - "Whole body within `unit_body_max`; no investigation narrative in the body"
    - "Every category gets an `applies_to` selector derived from the layer it governs (paths globs / phases from the closed set); a rule's own selector only narrows it"
    - "Digest line and index entry are written in the same edit as the heading"
    - "Every extracted lesson is a rule; a skill forms later from an accumulated rule cluster (Knowledge Scaling → Consolidation), not at extraction time"

  severity_levels:
    must: "Mandatory — violation blocks review. Core architectural constraints and naming conventions."
    should: "Recommended — violation triggers warning. Established patterns with occasional justified exceptions."
    prefer: "Advisory — preferred approach when no other constraints apply. Style preferences."

  extraction_procedure:
    step_1: "Read existing linting/formatting configs (.editorconfig, .eslintrc, .prettierrc, ruff.toml, checkstyle.xml, .golangci.yml)"
    step_2: "Analyze 5-10 representative files per module layer to identify recurring patterns"
    step_3: "Cross-reference patterns across layers — distinguish project-wide vs layer-specific rules"
    step_4: "For each pattern found in 3+ locations, create a rule entry with severity"
    step_5: "Include real code examples with file path references"
    step_6: "Flag inconsistencies found during extraction in issues.md"
    step_7: "Write rules documents per category"
    step_8: "Generate _index.yaml with all rules summarized"

  rules:
    - "Extract rules from ACTUAL code patterns, not assumptions"
    - "Every rule MUST have at least one real code example with file path"
    - "Severity must reflect observed enforcement level — if violations exist, use 'should' not 'must'"
    - "Rules apply to NEW code — do not require refactoring existing code"
    - "If a pattern is inconsistent across the codebase, document both variants and flag in issues.md"
    - "Do not invent rules not evidenced by the codebase"
    - "Keep rules actionable — a developer reading the rule should know exactly what to do"
    - "Group related rules in the same document, not scattered across files"
    - "Rules documents are living documents — updated during implement, review, and propagate phases"
}
```

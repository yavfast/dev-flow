# Conventions

Document statuses, versioning, traceable identifiers, the `docs/` file set, the `.dev_flow/rules/` layout, branch / PR mapping, entry routes.

## Document Status Vocabulary

```
draft -> active -> deprecated        (concepts, specifications — living documents)
draft -> in-progress -> completed    (plans — finite work items)
```

| Status | Meaning | Applies to |
|--------|---------|------------|
| `draft` | Initial version, not yet validated through gate | All |
| `active` | Validated and current | Concepts, Specifications |
| `in-progress` | Being actively worked on | Plans |
| `completed` | All work done | Plans |
| `deprecated` | Superseded or no longer relevant — excluded from conflict checks | All |

## Versioning

When a concept or specification undergoes a **breaking change** (incompatible contract changes, removed entities, fundamentally different approach), create a new version instead of editing in place:

1. Set old document to `Status: deprecated` with `Deprecated-reason: Replaced by [C_XXX_v2](./path)`
2. Create new document with version suffix: `C_XXX_v2`, `SP_XXX_v2`
3. Update all `Depends on` / `Used by` references in dependent documents
4. Both versions may coexist while dependents migrate

For **non-breaking changes** (adding fields, extending contracts, fixing descriptions), edit the existing document in place and update the Changelog (if maintained — see [Docs Scaling](docs-scaling.md)).

## Traceable Identifiers

Every section has a unique, immutable identifier:

| Document | Format | Example |
|----------|--------|---------|
| Concept | `C_XXX_NN_NN` | `C_ACS_01_01` |
| Specification | `SP_XXX_NN_NN` | `SP_ACS_01_01` |
| Plan | `PL_XXX` | `PL_ACS` |
| Epic | `E_XXX` | `E_ACM` |
| Design decision | `<DocID>_DEC_NN` | `C_ACS_DEC_01`, `SP_ACS_DEC_02`, `PL_ACS_DEC_01` |

The `_DEC_NN` form identifies a record in a document's **Design Decisions** section (see [Interview Mode](interview-mode.md)); the `_DEC` segment is reserved and never used as a numeric section number.

In code, reference these as comments: `# [C_ACS_03_01] PermissionInterceptor`

## File Organization

All documents live in `docs/`, follow [Documentation Formatting](formatting.md) (one paragraph = one logical line, no hard wraps inside sentences) and the `documentation` register of [Output Styles](output-styles.md). One concept = one file set:

| File | Extension | Example |
|------|-----------|---------|
| Spike | `*.spike.md` | `access_control.spike.md` |
| Concept | `*.concept.md` | `access_control.concept.md` |
| Specification | `*.sp.md` | `access_control.sp.md` |
| Plan | `*.plan.md` | `access_control.plan.md` |
| Epic | `*.epic.md` | `access_management.epic.md` |
| Split-set child | `*.p<NN>.plan.md` / `*.<module>.sp.md` | `access_control.p02.plan.md`, `access_control.audio.sp.md` |
| Index | `_index.md` | `docs/_index.md` |
| Glossary | `_glossary.md` | `docs/_glossary.md` |
| Framework map | `_framework.md` | `docs/_framework.md` |
| Adoption notes | `ext_adoption/*.md` | `docs/ext_adoption/other_repo.md` |

- **Umbrella sets.** A document split under [Docs Scaling](docs-scaling.md) becomes an umbrella (the original file name, the canonical entry point) plus child files; `_index.md` lists only umbrellas; no split renames or renumbers a traceable ID.
- **Spike** — optional pre-concept investigation from [research](../phases/research.md); passes no gate. **Epic** — optional grouping for a feature spanning 3+ closely related concepts; template [templates/epic.md](../templates/epic.md).
- **`_index.md`** — maintained once `docs/` exceeds 5 documents; the phase that creates a file adds its row, unasked; a router (one-line annotation per file + ID-prefix → file map), never a section catalogue ([Docs Scaling](docs-scaling.md)).
- **`_glossary.md`** — canonical domain vocabulary, created lazily, and **loaded alongside `_index.md` whenever present**, independent of the 5-doc threshold ([Glossary](glossary.md)).
- **`_framework.md`** — the living architectural map, maintained only by onboard and `audit code`, **loaded alongside `_index.md` on code-touch phases**; it links down to rules/skills and inlines none of their detail ([Code Audit](code-audit.md)).
- **`ext_adoption/`** — advisory analyses from [`adopt`](repo-adoption.md): no traceable ID, no gate, not a backlog; consulted by the concept phase's Reuse Check; excluded from the `audit docs` integrity checks.
- **Index format.** Machine-read catalogues (`.dev_flow/rules/`, `skills/`, `tools/`, `roles/`, `cache/`) use `_index.yaml`; human-browsed ones (`docs/`, `.dev_flow/tasks/`, `todos/`) use `_index.md`. Apply the split to any new collection.

## Project Rules

When `.dev_flow/rules/` exists, all new code MUST comply ([Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)). Rules are extracted during onboard and updated during implement/fix/review. Severity: **must** (blocks review) · **should** (warning) · **prefer** (advisory); rules apply to new code only.

```
.dev_flow/rules/
├── _index.yaml         # Derived router — regenerated from the files
├── naming.md · structure.md · architecture.md · error-handling.md · style.md · testing.md
```

Each rule is one h2 unit — directive and severity in the heading over a bounded body, immutable id; a category above `digest_min` opens with a `## Contents` digest; an optional `applies_to` selector is what the knowledge gate matches. A skill forms from an accumulated rule cluster at the agent's discretion. See [Knowledge Scaling](knowledge-scaling.md) and the [rule phase](../phases/rule.md).

**Auto-discovery (self-learning).** At the `implement` and `fix` reflection checkpoints ([Experience Capture](experience-capture.md)), a coding pattern, constraint, or violated invariant with no matching rule is **written automatically, without asking**: default severity `should` (`prefer` for advisory); **never an auto-`must`** — a `must`-strength lesson or a contradiction with an existing rule goes to an independent clean-context review first ([Delegation for Focus](delegation.md)). The rule phase writes the file and the index entry; the commit message names the rule; the developer reviews every auto-written rule in the commit diff — the commit sign-off is untouched. **Never a fact about the developer or the machine** — that belongs to `.dev_flow/profile/` under its consent regime, and a `declined` category is dropped rather than re-homed as a rule or skill ([Adaptation Profile](adaptation-profile.md)).

## Git Branch / PR Mapping

| Scope | Branch / PR |
|-------|-------------|
| Concept + Specification | One PR |
| Implementation Plan | Separate PR |
| Implementation (per plan phase) | One PR per plan phase |
| Review + Propagation | Same PR as the triggering change |

## Greenfield vs Takeover

- **Greenfield:** concept -> domain model -> specification -> plan -> code.
- **Takeover:** `/dev-flow onboard` — reverse-engineers docs and rules from the code, bottom-up and resumable: [onboard phase](../phases/onboard.md).

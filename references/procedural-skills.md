# Procedural Skills — Project Memory That Outranks a Stale Prior

Cross-cutting refinement of the **skill subsystem** ([C_PHK](../docs/phases-knowledge.concept.md)) — concept [C_SPM](../docs/skills_as_procedural_memory.concept.md) · spec [SP_SPM](../docs/skills_as_procedural_memory.sp.md). Not a pipeline stage and no command: it shapes the *form* of a `.dev_flow/skills/` entry and the *precedence* its retrieval carries, inside the existing lazy [skill check](../phases/skill.md). Skills are still **advisory, lazy, on-demand** — this never adds a development gate.

## Why this exists

An agent trusts its **training prior** over the project's reality — reaching for outdated practices, framework versions, and APIs even when the project already knows better. The fix is to let the project's own distilled experience win. But *forcing* all work through a skill backfires (confirmed by a [4-lens spike](../docs/skills_as_procedural_memory.spike.md)): mandatory, total, eager skill use breeds sprawl, a greenfield cold-start tax, retrieval bloat, and **cognitive entrenchment** (the agent rides a stored procedure and stops seeing a better one). So the lever is **structural pull, not a gate** — a *current* matching skill outranks the prior; a *stale* one is demoted until re-grounded.

## The form: a skill is procedural memory

A skill captures *how to use tool X here / how to do process Y here* — not a reference of facts derivable from the codebase (the [Non-Triviality Filter](../phases/skill.md#non-triviality-filter) stays the admission gate; **Minimality** stays — no skill without a stated future consumer). Additive fields on the skill file, over the C_PHK template:

| Field | Meaning |
|-------|---------|
| `kind: procedural` | This skill is a distilled procedure (the kind this reference governs). |
| `procedure` | The distilled *how-to* body — never a fact-reference. |
| `applicability_boundary` | **Mandatory:** `when_to_use` / `when_not` / `version_or_context_scope`. A skill with no boundary is a negative-transfer trap. |
| `freshness` | `written_against` (tool/framework version or context description) + `state: current\|stale`. **Optional** for a pure-process skill with no version. |
| `promotion` | `candidate` (born of one hard case — narrow) \| `established` (confirmed by a 2nd convergent use — generalized). |
| `scope_note` | Required while `candidate`: the observed context the candidate is limited to. |

## Retrieval precedence — the contract

Inside the existing lazy match (chain through `_index.yaml` — **never load the whole tree**):

1. **No match → never block.** Fall back to the prior + ordinary research (optionally capture a new skill afterward). "No skill" is the normal greenfield case — no tax.
2. **Current matching skill → authoritative over the general prior** for that tool/process. This is the whole point: the project's procedure beats possibly-stale memory.
3. **Stale matching skill → re-ground first.** A `stale` skill (version/context drifted past `written_against`) is **never applied blindly** and **cannot silently override fresh research**: cheaply re-ground (check installed version / official docs, or a short spike), re-stamp `written_against` + `state→current`, *then* rely on it. A confidently-applied stale procedure is worse than none.
4. **Candidate out of scope → no-op fallback.** A `candidate` applied outside its `scope_note` falls back to the prior, never a forced (wrong) reuse.

The agent's declarative reasoning never switches off — a loaded skill **shifts** it toward project truth, it does not **bind** it (no Einstellung lock; a repeated miss is a demotion/curation signal).

## Promotion & freshness (structural signals, never a self-score)

- **Birth & promotion** (auto-harvested by [Experience Capture](experience-capture.md), C_EXC DEC_06 — no permission prompt): a hard fix / tool-or-process recurrence ≥N / research consolidation that passes Non-Triviality → write/update the skill. One hard case → `candidate` scoped to its context; a **second convergent use** → `established` + generalize the boundary. (One case is a *case*, not yet a *schema* — mirrors "auto-write defaults to `should`, never `must`".)
- **Freshness** drives demotion: on version/context drift the skill goes `stale` (loses precedence over fresh research) until re-grounded + re-stamped → `current`. A re-stale of an `established` skill demotes like any other; promotion is not reset.

## Curation (audit `skills` scope)

[audit](../phases/audit.md) grooms the catalogue — **incrementally, never a bulk rewrite** (avoids the context-collapse / brevity-bias rot seen in prior art):

- `prune` low-hit / stale-beyond-use, `merge` near-duplicates → **propose** (destructive, like skill deletion — goes through review, consistent with C_EXC autonomy).
- `restamp` freshness, incremental edits → **apply**.
- **Conflict** (two skills prescribe different procedures for one surface) → **surface as an explicit decision; never let retrieval silently pick a side** (the least-mitigated failure in prior art).

## Where it is wired

| Touchpoint | What it carries |
|------------|-----------------|
| [skill phase](../phases/skill.md) | The procedural template fields + the retrieval-precedence contract |
| [SKILL.md → Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding) | The one-line precedence rule (current skill > prior; stale → re-ground) |
| skill-check in [implement](../phases/implement.md) / [fix](../phases/fix.md) / [testing](../phases/testing.md) / [verify](../phases/verify.md) | Precedence applies whenever a loaded skill matches the work |
| [audit](../phases/audit.md) `skills` scope | Curation: prune/merge propose · conflict surface · restamp/edit apply |
| [Experience Capture](experience-capture.md) | The birth form — procedure + mandatory boundary + freshness stamp; single hard case → candidate |

## Boundaries

- **Structural pull, not a gate.** "No matching skill" never blocks; the word *only* (as in "development only through skills") is explicitly rejected (C_SPM DEC_01).
- **Boundary is mandatory content.** A skill without an applicability boundary is rejected at write time.
- **Trust structure, not a self-rating.** Promotion/demotion follow observed recurrence and version/context drift — never a confidence number.
- **Re-firing the precedence contract per action belongs to [Application Enforcement](application-enforcement.md).** This reference fixes the skill's *form* and *precedence*; that contract is front-loaded and fades without per-burst re-triggering — which C_AEN owns. The freshness state set here is carried in its Pre-Action Marker.

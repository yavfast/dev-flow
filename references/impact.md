# Impact Walk — Blast Radius of a Change

Shared sub-procedure for the **Ask**, **Do**, **Propagate**, and **Review** phases (with optional uses in audit and interviews). It is **not** a standalone pipeline stage and has no command — invoke it from inside a phase whenever a decision needs to know *what a change would touch*. The natural user entry is `/dev-flow ask` ("what breaks if …?").

## Why this exists

"What does this change touch?" is the most repeated question in the pipeline: ask answers it for feasibility, do for change classes, propagate for cascades, review for conflicts and deprecation. Done by hand, every phase re-derives the answer by reading documents into context — and hand walks reliably miss edges: `Used by` is maintained by hand and drifts, and code bindings (traceable ID comments) are invisible from the documents. The result is the classic failure the skill exists to prevent: a spec changes, and a module bound to it surfaces broken at Verify — or in production.

The Impact Walk makes the answer deterministic and cheap: one bidirectional walk over metadata that already exists — `Depends on` / `Used by`, cross-type fields, `Implements:` references, traceable IDs in code — returning the **radius, not the documents**.

## Input

One starting point:

- a **traceable ID** (`C_XXX[_NN_NN]`, `SP_XXX[_NN_NN]`, `PL_XXX`) — most precise;
- a **document path** — resolves to its `Code:` ID;
- a **code file** — resolves to the IDs in its traceable comments.

And a direction, defaulting to both:

- **downstream** ("who consumes this?") — for changing or deprecating the node;
- **upstream** ("what does this rest on?") — for understanding what it inherits.

## The walk

1. **Resolve the start node** to one or more traceable IDs.
2. **Doc edges, both directions.** From the node's document: `Depends on` / `Used by` metadata, the cross-type fields (`Concept:` / `Specification:` / `Plan:`), and in-body section references (`Implements: [C_XXX_NN]`, `Uses [SP_YYY_NN_NN]`). Walk transitively; record depth.
3. **Code bindings.** Grep the codebase for the affected IDs (the `[C_XXX`, `[SP_XXX`, `[PL_XXX` comment forms) — collect file:line per ID. Include test files; they bind through the same IDs.
4. **Active work.** Scan `.dev_flow/active_context.md` (with `tasks/_index.md` as the cheap first pass), then the matching task files' Current Work Item / Relevant Context for the affected IDs — an in-flight task inside the radius is a coordination risk, not just another doc edit.
5. **Asymmetries are findings.** A depends on B but B's `Used by` misses A; an ID in code with no defining document; a dependency on a `deprecated` document — collect these as metadata-drift findings. Do **not** fix them mid-walk; route them to [propagate](../phases/propagate.md).

## Output — the radius, not the dumps

Report compactly, grouped by layer, with depth and counts:

```
Impact of SP_AUTH_02_01 (downstream):
  Documents (4):
    SP_SESSION   depends on SP_AUTH               docs/session.sp.md
    PL_AUTH      implements                       docs/auth.plan.md
    C_GATEWAY    integration point (depth 2)      docs/gateway.concept.md
    ...
  Code (14 files): src/auth/* (9), src/gateway/middleware.* , tests (4 files)
  Active tasks (1): task_C_GATEWAY — in-progress (coordination risk)
  Drift findings (2): SP_SESSION missing from SP_AUTH "Used by";
                      orphan ID [SP_AUTH_03_01] in src/legacy/token.*
  Summary: 4 docs / 14 code files / 1 active task
           cascade > 3 docs — reconsider boundaries? (propagate rule)
```

The counts feed the consuming phase's thresholds (propagate's ">3 documents" boundary check, do's change classes, ask's scope estimate). Raw grep output stays out of the report.

## Where it is consumed

| Consumer | Use |
|----------|-----|
| [ask](../phases/ask.md) | The procedure behind the "Impact assessment" question type — and the natural user entry ("what breaks if …?") |
| [do](../phases/do.md) | Evidence for Change Classes when trivial/standard/architectural is unclear |
| [propagate](../phases/propagate.md) | Cascade Impact Assessment; the dependents to-do list for versioning and deprecation |
| [review](../phases/review.md) | Enumerating dependents during conflict detection and deprecation checks |
| [audit](../phases/audit.md) | *Optional:* intersecting two active tasks' radii to flag overlapping work |
| [Interview Mode](interview-mode.md) | *Optional:* numbers for an option's Consequence line when a fork's cost is its blast radius |

## Boundaries

- **Read-only.** The walk changes nothing; the drift it surfaces is routed to [propagate](../phases/propagate.md).
- **As good as the metadata.** Missing `Used by` rows and uncommented code shrink the radius silently — which is exactly why asymmetry findings are part of the output: each run repairs the map a little, improving the next run.
- **Delegate the grep.** The walk is search-heavy and floods context — a textbook [focus delegation](delegation.md): the helper returns the radius and the findings; full match lists stay in a workspace file referenced by path.
- **Same machinery as drift detection.** The [drift detection algorithm](../phases/propagate.md#drift-detection-algorithm) collects the same IDs globally (orphans, stale docs); the Impact Walk queries one node's neighborhood. Reuse the collection approach — don't maintain two.

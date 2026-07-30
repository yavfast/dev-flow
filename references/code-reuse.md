# Code Reuse — Search Before Create, Build For Reuse

Cross-cutting sub-procedure of the **code-writing** phases — [implement](../phases/implement.md) and [fix](../phases/fix.md) (a *glance* from [plan](../phases/plan.md) when it names shared utilities). It is **not** a standalone pipeline stage and has no command — an **advisory** discipline applied at the moment code is written. It adds no new pass/fail gate; it informs the code the phase was already going to write.

## Why this exists

Reuse is settled once, high up: the concept phase's [Reuse Check](../phases/concept.md#reuse-check) prevents a *duplicate concept*. Nothing carries that discipline down to where functions and classes are actually typed — so two failure modes leak through: **duplication in** (re-implementing a helper/validator/constant that already exists because nobody searched) and **duplication later** (writing single-use code a near consumer then copy-pastes). This closes the gap at code altitude, per-action-burst, the way [Application Enforcement](application-enforcement.md) re-triggers the knowledge gate.

## The discipline

Each half is scaled by [change-class](../phases/do.md#change-classes) — skip for a trivial one-liner.

### 1. Search before create

Before writing a new function, class, method, util, or shared constant, look for an existing one — cheapest source first:

1. **`docs/_framework.md`** shared-utilities / extension-points map (loaded on code-touch phases) and **`.dev_flow/skills/`** — the project's own catalogue of where things live.
2. **The codebase** — symbol/definition search and a content grep for the behaviour (not just the name you'd pick). Delegate a *wide* search to a focus helper and keep the conclusion, not the dump ([Delegation for Focus](delegation.md)).

Then:
- **Exact equivalent exists** → call it. Do not re-implement.
- **Near-equivalent exists** → extend/parameterize it rather than fork a copy — **unless** the two cases change for genuinely different reasons (see restraint below).
- **Reuse needs a contract change** to the existing symbol → do not silently widen it and do not fork around it: treat the owning contract as upstream ([Upstream Escalation](escalation.md)).
- **Nothing exists** → create it, and apply half 2.

### 2. Build for reuse — YAGNI-gated

When the code you are about to write has a *known or near-term* second consumer, leave a clean reuse seam: name it for the behaviour not the caller, keep it dependency-light, and place it at the right layer per `docs/_framework.md`. But gate every "make it general" impulse through the [Consequence Forecasting](consequence-forecasting.md) YAGNI-gate — the gate is **strict** at code altitude:
- a **real, near-term** consumer → `build now` (extract the seam);
- a **plausible but untriggered** one → `seam + flag` at most (a thin extension point, no speculative machinery);
- **no trigger** → `drop + record` — write the direct, single-use version; do not pre-abstract.

This never overrides [Minimality](../SKILL.md#validation-gates): a reuse seam with no stated consumer is itself dead weight.

## Restraint — resist over-DRY

Two snippets that merely *look* alike are not a reuse target if they **change for different reasons** — folding them couples unrelated concerns and creates a worse problem than the duplication. Same shape ≠ same responsibility. This mirrors the `duplication` lens's over-DRY caveat in [Code Audit](code-audit.md).

## Relationship to existing mechanisms

- **Concept [Reuse Check](../phases/concept.md#reuse-check)** is the design-altitude, once-per-concept twin; this is its code-altitude, per-burst counterpart.
- **`audit code` `duplication` lens** ([Code Audit](code-audit.md)) is the *retrospective* backstop that finds duplication already shipped; this prevents it at write time.
- **Realized as a Pre-Action Marker** — the search-first / seam-with-trigger reminder rides the per-burst marker of [Application Enforcement](application-enforcement.md); it is a pointer, not a separate load.

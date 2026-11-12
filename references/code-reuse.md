# Code Reuse — Search Before Create, Build For Reuse

Cross-cutting sub-procedure of the **code-writing** phases — [implement](../phases/implement.md) and [fix](../phases/fix.md) (a *glance* from [plan](../phases/plan.md) when it names shared utilities); [review](../phases/review.md) checks it through its `Code reuse` row. It is **not** a standalone pipeline stage and has no command — an **advisory** discipline applied at the moment code is written. It adds no new pass/fail gate; it informs the code the phase was already going to write.

## The discipline

Each half is scaled by [change-class](../phases/do.md#change-classes) — skip for a trivial one-liner.

### 1. Search before create

Before writing a new function, class, method, util, or shared constant, look for an existing one — cheapest source first:

1. **`docs/_framework.md`** shared-utilities / extension-points map (loaded on code-touch phases) and **`.dev_flow/skills/`** — the project's own catalogue of where things live.
2. **The codebase** — symbol/definition search and a content grep for the behaviour (not just the name you'd pick). Delegate a *wide* search to a focus helper and keep the conclusion, not the dump ([Delegation for Focus](delegation.md)).
3. **The standard library and the platform** — a language, framework, or platform feature that already does it.
4. **An already-installed dependency.**

Before hand-rolling a **check or operation** (not a code symbol), search the project tool router `.dev_flow/tools/_index.yaml` the same way: a tool whose range of use covers the need → use it; a near case that changes for the same reason → `extend` it; nothing → write the script in the session workspace and, if it proves worth reusing, [promote](project-tools.md#promote) it. A tool's range of use is bounded by the YAGNI gate below.

Then:
- **Exact equivalent exists** → call it. Do not re-implement.
- **Near-equivalent exists** → extend/parameterize it rather than fork a copy — **unless** the two cases change for genuinely different reasons (see restraint below).
- **Reuse needs a contract change** to the existing symbol → do not silently widen it and do not fork around it: treat the owning contract as upstream ([Upstream Escalation](escalation.md)).
- **Nothing exists** → create it, and apply half 2. Never add a new dependency for what a few lines do; a new dependency is a [plan](../phases/plan.md) technology decision.

### 2. Build for reuse — YAGNI-gated

When the code you are about to write has a *known or near-term* second consumer, leave a clean reuse seam: name it for the behaviour not the caller, keep it dependency-light, and place it at the right layer per `docs/_framework.md`. But gate every "make it general" impulse through the [Consequence Forecasting](consequence-forecasting.md) YAGNI-gate — the gate is **strict** at code altitude:
- a **real, near-term** consumer → `build now` (extract the seam);
- a **plausible but untriggered** one → `seam + flag` at most (a thin extension point, no speculative machinery);
- **no trigger** → `drop + record` — write the direct, single-use version; do not pre-abstract.

This never overrides [Minimality](../SKILL.md#validation-gates): a reuse seam with no stated consumer is itself dead weight.

## Restraint — resist over-DRY

Two snippets that merely *look* alike are not a reuse target if they **change for different reasons** — folding them couples unrelated concerns and creates a worse problem than the duplication. Same shape ≠ same responsibility.

## Relationship to existing mechanisms

- **Concept [Reuse Check](../phases/concept.md#reuse-check)** is the design-altitude, once-per-concept twin; this is its code-altitude, per-burst counterpart.
- **`audit code` `duplication` lens** ([Code Audit](code-audit.md)) is the *retrospective* backstop that finds duplication already shipped; this prevents it at write time.
- **Realized as a Pre-Action Marker** — the search-first / seam-with-trigger reminder rides the per-burst marker of [Application Enforcement](application-enforcement.md); it is a pointer, not a separate load.

# Application Enforcement — Re-Trigger Knowledge at the Moment of Action

Cross-cutting sub-procedure for the **knowledge-touch** phases — concept [C_AEN](../docs/application_enforcement.concept.md) · spec [SP_AEN](../docs/application_enforcement.sp.md). It moves the [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding) gate from *phase start* (once, C_PHK) to *the moment of action* (per action burst). It is **advisory at its core** and never a standalone stage; the harder tiers engage only for high-stakes work, and only where the runtime can observe an action.

## Why this exists

dev-flow gates the *loading* of knowledge (MUST read/load at phase start) and barely gates its *application*. But **loaded ≠ salient ≠ applied**: in a long session a rule drifts from attention (lost-in-the-middle / context rot), vanishes on compaction, and one "classic" action seeds a self-imitation loop the agent **does not recover from** (the [procedural_rule_enforcement spike](../docs/procedural_rule_enforcement.spike.md), grounding the [[loaded-not-applied]] observation). A front-loaded gate fades exactly when the action repeats. So enforcement must be **bound to the action, not the phase start**, and **as external as possible** — because the acting agent is the thing that drifts.

## The discipline: activate → mark → act → check → (escalate)

Per **action burst** — a series of calls of one action type (e.g. "test run"), **not** each tool call (per-call is noise + the overthinking inverted-U):

1. **Activate knowledge.** Determine the small relevant set: from the plan's **Required Knowledge** (standard flow) or a lazy `_index.yaml` match (other flows — weaker fallback). Empty set → no marker, proceed (no friction).
2. **Emit a Pre-Action Marker** — *names* of applicable rules/skills + their freshness state (from [Procedural Skills](procedural-skills.md)), **pointer-only** (never the full body — that bloat is what feeds drift) and **fact-only** (no subjective self-score). Place it adjacent to the burst for recency. **Carrier:** a short inline note — the **pre-action sibling of the [Response Trailer](experience-capture.md#the-response-trailer--response-granularity-carrier)** (that one is post-hoc; this is pre-action), sharing its fact-only format. (SP_AEN DEC_02 resolved → inline; a runtime that supports message meta-tags may use one instead.)
3. **Act** (the burst).
4. **Conformance tripwire** — a cheap **deterministic, non-model** check that the action went through the sanctioned path. The signal is a non-model by-product (`exit-code` / `captured-command` / `diff` / `log`), never the agent's own prose (else it Goodhart-games into ritual). Off-path → **flag, not silently**.

## Enforcement tiers (proportional to stakes × checkability × runtime)

| Tier | When | Runtime |
|------|------|---------|
| `advisory` | Default — re-trigger + marker. Enough for most actions. | **No** runtime needed (it is just a message). |
| `interface-gate` | High-stakes **and** the procedure reduces to a sanctioned entry **and** the runtime can intercept/observe the action. Out-of-band action fails or is detected by the tripwire — converts a semantic (un-hookable) substitution into a structural, hookable one. Strongest lever *where available*; each reduced slice shrinks residue. | **Requires** a runtime that observes the action (hook / sanctioned runner). |
| `sampled-verifier` | High-stakes where the gate does not reduce: a **fresh** agent judges conformance to the documented procedure as a **rubric**, **cross-model**, at a phase gate / sample / tripwire escalation. | Requires a fresh cross-model agent ([Delegation](delegation.md)). |

**Self-attestation by the acting agent is excluded as a control** — "did I comply?" checked by the same drifting agent is theatre (Huang 2310.01798: self-correction without an external signal is unreliable; a same-family / free-form judge rubber-stamps drift). Verification is always a deterministic tripwire or a *fresh* cross-model rubric verifier, **never per-run**.

## Runtime precondition & graceful degradation

`advisory` + the Pre-Action Marker work **everywhere** — they only re-surface knowledge. `interface-gate` and the tripwire **require** `runtime_can_observe()` (a hook-like capability), exactly as [Experience Capture](experience-capture.md) leans on window-fill *where the runtime exposes it*. Where that capability is absent, these tiers **degrade to `advisory` + `sampled-verifier`** (`NO_RUNTIME`) — they do not silently "guarantee" enforcement and they do not break. The concept does not promise enforcement on its own; it is a runtime-conditional dependency.

## Drift-awareness (structural triggers, never self-assessment)

Activation is fixed per-burst, but tripwire **sensitivity** and verifier **sampling** rise on structural drift signals — priority to the **first classic action** (the loop does not recover, so catch the first), then post-compaction, session growth, prior-divergence. Modulation only raises sample/sensitivity within the already-chosen tier; it adds no enforcement of its own, and is derived from deterministic events — never a "feels like I'm drifting" self-rating.

## Where it is wired

A terse **"Knowledge activation"** line sits in each knowledge-touch phase, pointing here:

| Touchpoint | What it carries |
|------------|-----------------|
| [implement](../phases/implement.md) / [fix](../phases/fix.md) / [testing](../phases/testing.md) / [verify](../phases/verify.md) | A **"Knowledge activation"** gate-line — per-burst re-trigger + Pre-Action Marker; tripwire/tier where the burst is high-stakes |
| [review](../phases/review.md) | The clean-context pre-commit review **realizes the `sampled-verifier` tier** — a fresh, cross-model agent judging conformance to documented procedure; it is the external verifier, not a drift site needing re-trigger |
| [plan phase](../phases/plan.md) + [plan template](../templates/plan.md) | The **Required Knowledge** section — the source of truth for the activation set in the standard flow |
| [SKILL.md → Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding) | The gate is re-triggered per burst at the moment of action, not once at phase start |
| [Procedural Skills](procedural-skills.md) | The marker carries each skill's freshness state; the per-burst re-trigger is what keeps "current skill > prior" from fading |
| [Experience Capture](experience-capture.md) | The Pre-Action Marker shares the Response Trailer's fact-only format (pre-action vs post-hoc siblings) |

## Boundaries

- **Scope = all project-knowledge** — rules, skills, relevant memory — not skills alone (a formatting rule is a *rule*; a non-standard test procedure is a skill+rule). Narrowing to skills would re-open the gap on rules (DEC_05).
- **Re-trigger is advisory, not blocking.** It raises adherence; only the structural tiers (tripwire / interface-gate) give a guarantee, and only where the runtime allows.
- **Pointer-only, fact-only, per-burst.** A marker with a full body or a self-score is rejected; activation per-call is forbidden.
- **Honest residue.** Judgment-shaped conformance with no sanctioned entry is not auto-guaranteed — only sampled, bounded assurance + a tripwire on artifact shape. The real fix is reducing residue via interface-gating, not leaning harder on the verifier. Track a false-positive budget; rubric over free-form.

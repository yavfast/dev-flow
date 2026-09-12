# Application Enforcement — Re-Trigger Knowledge at the Moment of Action

Cross-cutting sub-procedure for the **knowledge-touch** phases. It moves the [Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding) gate from *phase start* (once) to *the moment of action* (per action burst). It is **advisory at its core** and never a standalone stage; the harder tiers engage only for high-stakes work, and only where the runtime can observe an action.

## Why this exists

**Loaded ≠ salient ≠ applied.** A front-loaded gate fades exactly when the action repeats, so enforcement binds to the action and stays as external as the runtime allows.

## The discipline: activate → mark → act → check → (escalate)

Per **action burst** — a series of calls of one action type (e.g. "test run"), **not** each tool call (per-call is noise):

1. **Activate knowledge.** Determine the small relevant set: from the plan's **Required Knowledge** (standard flow) or the selector-computed relevant set of [Knowledge Scaling](knowledge-scaling.md) (other flows — category and unit `applies_to` matched against the touched paths and the current phase, always-on categories included; directives only, a body on a trigger; over `activation_budget` drops nothing and lowers no severity). Empty set → no marker, proceed (no friction).
2. **Emit a Pre-Action Marker** — *names* of applicable rules/skills + their freshness state (from [Procedural Skills](procedural-skills.md)), **pointer-only** (never the full body — that bloat is what feeds drift) and **fact-only** (no subjective self-score). Place it adjacent to the burst for recency. **Carrier:** a short inline note — the **pre-action sibling of the [Response Trailer](experience-capture.md#the-response-trailer--response-granularity-carrier)** (that one is post-hoc; this is pre-action), sharing its fact-only format. (Inline; a runtime that supports message meta-tags may use one instead.)
3. **Act** (the burst).
4. **Conformance tripwire** — a cheap **deterministic, non-model** check that the action went through the sanctioned path. The signal is a non-model by-product (`exit-code` / `captured-command` / `diff` / `log`), never the agent's own prose. Off-path → **flag, not silently**. On-path, record the same by-product as the primary source of the `exercised` evidence state ([Evidence Discipline](evidence-discipline.md)).

## Enforcement tiers (proportional to stakes × checkability × runtime)

| Tier | When | Runtime |
|------|------|---------|
| `advisory` | Default — re-trigger + marker. Enough for most actions. | **No** runtime needed (it is just a message). |
| `interface-gate` | High-stakes **and** the procedure reduces to a sanctioned entry **and** the runtime can intercept/observe the action. Out-of-band action fails or is detected by the tripwire. Strongest lever *where available*. | **Requires** a runtime that observes the action (hook / sanctioned runner). |
| `sampled-verifier` | High-stakes where the gate does not reduce: a **fresh** agent judges conformance to the documented procedure as a **rubric**, **cross-model**, at a phase gate / sample / tripwire escalation. Fed the **artifact + contract only — never the acting agent's conclusion/justification**; prompted **adversarially** ("find what violates the procedure", not "is it good"). | Requires a fresh cross-model agent ([Delegation](delegation.md)). |

**Self-attestation by the acting agent is excluded as a control** — "did I comply?" checked by the same drifting agent is theatre. Verification is always a deterministic tripwire or a *fresh* cross-model rubric verifier, **never per-run**.

## Runtime precondition & graceful degradation

`advisory` + the Pre-Action Marker work **everywhere** — they only re-surface knowledge. `interface-gate` and the tripwire **require** `runtime_can_observe()` (a hook-like capability). Where that capability is absent, these tiers **degrade to `advisory` + `sampled-verifier`** (`NO_RUNTIME`).

## Drift-awareness (structural triggers, never self-assessment)

Activation is fixed per-burst, but tripwire **sensitivity** and verifier **sampling** rise on structural drift signals — priority to the **first classic action** (the loop does not recover, so catch the first), then post-compaction, session growth, prior-divergence. Modulation only raises sample/sensitivity within the already-chosen tier; it adds no enforcement of its own, and is derived from deterministic events — never a "feels like I'm drifting" self-rating.

One more signal targets the verifier itself — **verifier-rubber-stamp**: 2+ consecutive substantive reviews with zero actionable findings (a count, not a self-rating). A higher sample rate cannot fix a verifier that *is* the problem, so this one escalates — surface to the user; switch model if available.

## Where it is wired

A terse **"Knowledge activation"** line sits in each knowledge-touch phase, pointing here:

| Touchpoint | What it carries |
|------------|-----------------|
| [implement](../phases/implement.md) / [fix](../phases/fix.md) / [testing](../phases/testing.md) / [verify](../phases/verify.md) | A **"Knowledge activation"** gate-line — per-burst re-trigger + Pre-Action Marker; tripwire/tier where the burst is high-stakes |
| [review](../phases/review.md) | The clean-context pre-commit review **realizes the `sampled-verifier` tier** (per its row above — artifact+contract, adversarial); it is the external verifier, not a drift site needing re-trigger |
| [plan phase](../phases/plan.md) + [plan template](../templates/plan.md) | The **Required Knowledge** section — the source of truth for the activation set in the standard flow |
| [SKILL.md → Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding) | The gate is re-triggered per burst at the moment of action, not once at phase start |
| [Procedural Skills](procedural-skills.md) | The marker carries each skill's freshness state; the per-burst re-trigger is what keeps "current skill > prior" from fading |
| [Experience Capture](experience-capture.md) | The Pre-Action Marker shares the Response Trailer's fact-only format (pre-action vs post-hoc siblings) |

## Boundaries

- **Scope = all project-knowledge** — rules, skills, relevant memory — not skills alone (a formatting rule is a *rule*; a non-standard test procedure is a skill+rule).
- **Re-trigger is advisory, not blocking.** It raises adherence; only the structural tiers (tripwire / interface-gate) give a guarantee, and only where the runtime allows.
- **Pointer-only, fact-only, per-burst.** A marker with a full body or a self-score is rejected; activation per-call is forbidden.
- **Honest residue.** Judgment-shaped conformance with no sanctioned entry is not auto-guaranteed — only sampled, bounded assurance + a tripwire on artifact shape. The real fix is reducing residue via interface-gating, not leaning harder on the verifier. Track a false-positive budget; rubric over free-form.

# Consequence Forecasting — Phase-Scaled Lookahead + YAGNI-gate

Cross-cutting sub-procedure for the **decision-making** phases — concept, spec, plan, implement, fix. It is **not** a standalone pipeline stage and has no command — it is an **advisory** discipline invoked inside a phase whenever a *material* decision is about to be made. It does not add a new pass/fail gate; it informs the decision the phase was already making.

## Why this exists

Agents decide locally-correct but future-blind: each step looks right alone, yet the sum drifts — either **myopically** (a need missed now, costly to retrofit later) or to the opposite extreme, **speculatively** (building for an imagined future with no trigger — gold-plating). This is measured, not assumed: a research spike + live A/B verify found consequence-forecasting helps **non-monotonically** — a bare "look ahead" instruction surfaced a genuinely missed need in one task and produced over-engineering in another (1–1, no clean win). The governing variable is **not how far you look but the discipline around it**: scale the look to the phase, and gate every anticipation so it becomes a *seam or a record*, not premature work.

## The discipline: forecast → YAGNI-gate → decide

At a material decision point (skip trivial changes — scale by [change-class](../phases/do.md#change-classes)):

1. **Determine altitude from the phase** (table below) — never chosen freely.
2. **Forecast** the likely future needs / consumers / use-methods / scale / consequences at that altitude. Keep a thin *adjacent glance* (one altitude over): at `concept` glance **down** (is it feasible?), at `implement` glance **up** (does this still serve the intent?). Reuse [Impact Walk](impact.md) to estimate an anticipation's blast-radius — don't re-derive it.
3. **Gate each anticipation** through the YAGNI-gate → `build now` / `seam + flag` / `drop + record`.
4. **Decide**, informed by the verdicts. A genuinely uncertain build-vs-defer fork goes to [Interview Mode](interview-mode.md); a forecast that contradicts a *settled* higher assumption goes to [Upstream Escalation](escalation.md) — fix the document, don't bend the decision.

## Forecasting Altitude — set by the phase

Descending the pipeline: **scope narrows, fidelity sharpens, gate strictness rises** (monotonic). A coarse high-altitude forecast gives *direction*; a precise low-altitude one gives *correctness*.

| Phase | Scope | Horizon — what you forecast | Gate strictness | Typical verdict bias |
|-------|-------|-----------------------------|-----------------|----------------------|
| **concept** | whole project + intents/use-areas | feature lifecycle, evolution | lenient | direction, `seam`, open-decision (cheap — nothing is built) |
| **spec** | architecture, models, API | contract consumers; what must not break | moderate | contracts, versioning seams |
| **plan** | modules/classes, dependencies | which phase unblocks which; cost of a class change | moderate | phase order, boundaries |
| **implement** / **fix** | functions, blocks, tool-calls | blast-radius of the next steps; tool-call result | strict | correctness of *this* change; `drop` speculative future-proofing |

## The YAGNI-gate

Each anticipation gets exactly one verdict. Discriminators, cheap (don't expand into an analysis tree — that is the overthinking failure mode):

- **Is there a named, near-term trigger?** (a concrete consumer / requirement / event) — **no trigger → never `build now`** (this is the [Minimality gate](../SKILL.md#validation-gates): no consumer → cut).
- **Cost asymmetry:** is retrofitting later more expensive than acting now?
- **Seam or commitment?** A reversible *seam* (interface, extension point, nullable boundary) is cheap — take it freely. A *commitment* (imposes a present design tax, constrains the future) needs a trigger. A seam counts as "free" only when it costs ≈nothing **and** improves the code in front of you now; at strict (implement/fix) altitude a seam that exists *solely* for an untriggered future is the speculation the gate drops, not a free affordance.

| Verdict | When | Action |
|---------|------|--------|
| **build now** | named near-term trigger **and** deferral-cost > build-cost | build it, scaled by change-class |
| **seam + flag** | plausible, no near-term trigger, but a cheap reversible affordance saves a costly retrofit | leave the seam **+ record** a backlog item / open `DEC_NN` with a return trigger |
| **drop + record** | speculative; no trigger; the seam itself costs or constrains | don't build; **record** the considered-and-declined note with a return trigger |

**No silent deferral.** Every `seam`/`drop` leaves a record (backlog with trigger / open decision / a logged note) — "not now" is never lost. Strictness rises as altitude lowers: at `implement`, speculative future-need builds default to `drop`.

Worked contrast (from the verify): a URL shortener's **abuse handling** has a day-one trigger (public links) and a costly retrofit → **build now**; a **multi-tenancy** posture on an email-on-signup launch has no near-term trigger and imposes a present tax → **drop + record**.

## The one-step check (lowest altitude, free)

Before any non-trivial action — including when the fuller forecast was skipped for a trivial change — ask: *what is the inevitable next step, and does it undo or absorb this action?* If yes, the action is self-cancelling friction — drop or replace it. (Example: creating a branch whose only sequel is an immediate fast-forward merge + delete.) "Self-cancelling" means **no surviving effect** — if the cancelling step still *buys* something real (a review gate, a CI run, isolation a teammate relies on), the action is not self-cancelling and stands. This is the YAGNI-gate collapsed to a single free check.

## Where it is wired

An advisory **"Forecast check"** line sits in each decision-making phase, pointing here:

| Phase | What the forecast informs |
|-------|---------------------------|
| [concept](../phases/concept.md) | future needs/uses shaping the architecture; deep but coarse, gate lenient |
| [specification](../phases/specification.md) | contract consumers and what must not break |
| [plan](../phases/plan.md) | phase ordering and the cost of a structural change |
| [implement](../phases/implement.md) | blast-radius of the change + tool-call results; speculative future-proofing dropped |
| [fix](../phases/fix.md) | consequences of the fix; the one-step check on the diagnosis loop |

**Not** review or verify: those are *checking* phases, not decision phases — adding a forecast there is the scope creep this discipline teaches you to `drop`.

## Boundaries

- **Advisory, not a gate.** It never blocks a phase; it improves the decision the phase makes. It runs only at *material* decisions — trivial changes keep just the one-step check (no ceremony).
- **Distinct from [Impact Walk](impact.md), and composes it.** Impact Walk is **reactive** — the blast-radius of an *already-decided* change across the *existing* doc/code/task graph. Consequence forecasting is **proactive** — *hypothetical* futures shaping a *not-yet-made* decision. Forecasting *calls* Impact Walk for blast-radius; it does not duplicate it.
- **Credit foresight that changes a decision, not a ritual list.** An explicit "anticipated uses" section that alters nothing earns nothing; a capable agent's good decision already encodes foresight implicitly (verify finding).
- **Stay bounded.** A fixed, small set of discriminator questions per anticipation — never an open-ended hypothetical tree. More deliberation past the point of decision *reduces* quality (the overthinking inverted-U).
- **Harvest.** Recurring deferrals and what a trigger taught become rules/skills via [Experience Capture](experience-capture.md) — auto-applied through the structural gate (no permission prompt; would-be `must`/contradictions go to independent review).

# Experience Capture — Reflection & Salience at Transitions

Cross-cutting sub-procedure for every phase. Like [Delegation for Focus](delegation.md), it is **not** a pipeline stage — it runs *inside* whatever phase is executing, firing at the boundaries between stretches of work. Where delegation protects *attention* by moving noise out of the context, Experience Capture protects *memory*: it distills what a stretch of work produced — the decision, the lesson, the one line worth keeping — **before** the raw turns that produced it are compacted away.

## Contents

- [Why this exists](#why-this-exists) — Design commitments: store the distillation, not the transcript
- [The unit: a segment](#the-unit-a-segment) — What a segment is and where it ends
- [Transition Checkpoint — the primary cadence](#transition-checkpoint--the-primary-cadence) — Detection signals and the checkpoint steps: summary, re-grade, harvest
- [Harvest before demote (ordering invariant)](#harvest-before-demote-ordering-invariant) — Reflection runs before `pin` markers are demoted at task close
- [Applying experience — auto-apply through structural gates](#applying-experience--auto-apply-through-structural-gates) — Coverage rung, owner menu (`rule` · `skill` · `tool` · `pointer` · `access` · `improvement` · `drop`), checkability rung (mechanical → tool, judgement → rule), sweep gaps (a spec escalation → a sweep question), tool promotion ([Project Tools](project-tools.md)), doubtful writes → independent review
- [The Response Trailer — response-granularity carrier](#the-response-trailer--response-granularity-carrier) — Fact-only fields recorded per complex response
- [Context pressure — a tiered, proxy-driven trigger](#context-pressure--a-tiered-proxy-driven-trigger) — Proxy tiers and their responses; handoff, never auto-compact
- [Relation to working memory](#relation-to-working-memory) — What the checkpoint promotes from L1 to L2
- [Where it is wired](#where-it-is-wired) — The phases and steps that invoke this procedure
- [A note on these guidelines](#a-note-on-these-guidelines) — A projection, not a formal protocol: heuristics applied with judgment

## Why this exists

Design commitments:

- **Store the distillation, not the transcript.** What survives is a short summary plus any durable lesson — never the raw turns, which are referenced by path if kept at all.
- **Trust structure, not self-assessment.** Reflection fires on **deterministic structural events** dev-flow already logs, and salience is recorded with the **discrete markers** of [Salience Markers](../phases/status.md#salience-markers) — never a self-scored number ("importance 8/10", "confidence 90%").

North star: durable task state stays **complete in L2** (`.dev_flow/`) so the transcript is disposable and a fresh session restarts from files deterministically (see the [memory tiers](cache.md#memory--data-tiers-l0l1l2)).

## The unit: a segment

A **segment** is a coherent stretch of work on one topic or task, bounded by two transitions — e.g. "spec-SP_XXX" from opening the spec to finishing it. It has a short label, an owning task, and a lifecycle of exactly `open → closed` (closed once, by the transition that ends it). The label is what a response's trailer names; the checkpoint at its closing boundary is what distills it.

## Transition Checkpoint — the primary cadence

At a transition, run a **checkpoint** over the segment that is closing.

**Detecting a transition.** Anchor on **structural** signals — `phase-boundary`, `task-switch`, `subtask-done`, and the task close when no checkpoint ran over its last segment — which are deterministic events dev-flow already records. **Semantic** signals (a visible topic-shift, an explicit marker you raise) are a best-effort secondary: useful for an intra-phase checkpoint, but **never the sole basis when a structural signal is also available**, and self-detected topic boundaries alone are never a valid trigger (that is the self-assessment trap again).

**What a checkpoint does**, in order:

1. **Distill** the closing segment — a noisy read, so delegate it (see [Delegation for Focus](delegation.md)) and keep only the conclusion.
2. **Record a segment summary** as a Shared Activity Log entry tagged `{s:pin}` — a few lines of "what happened / what was decided", referencing any raw material by path, never inlining it (the no-large-blobs rule).
3. **Demote the raw turns** of the segment — re-grade them `{s:noise}`, or `{s:superseded→<summary>}` when the summary replaces them.
4. **Apply durable lessons** (next section) — auto-written through structural gates; a doubtful write goes to independent review first. A cause of delay the [Delay watch](delivery-focus.md#delay-watch) recorded — the agent's own or a delegate's `Friction:` item — is such a lesson when its step 3 says so. An escalation to a specification is one too, routed by [Sweep gaps](#sweep-gaps).
5. **Close the segment**, and **promote** the durable part of working memory to L2 (see [Relation to working memory](#relation-to-working-memory)).

## Harvest before demote (ordering invariant)

When a task **closes**, `DemoteOnTaskClose` makes its salience markers inert (a `pin` is task-scoped — see [Salience Markers](../phases/status.md#salience-markers)), re-grading the segment's still-pinned raw turns to effective `normal` and so making them eviction-eligible. The checkpoint that harvests the closing task's lesson MUST run **before** that demotion — reflect first, demote second — so reflection reads the segment's full, still-retained material before it can be evicted (and so the new summary's `pin` is recorded while the task is still active). The harvested lesson itself does not depend on the `pin` — it is written out through the structural rule/skill gate — but the reflection that produces it must run while its source is still protected. (This task-close *demote* — inerting all the task's markers — is distinct from the intra-checkpoint *demote* in step 3, which re-grades the segment's raw turns to `noise`.)

## Applying experience — auto-apply through structural gates

Self-learning is a **standing, ungated process**: a harvested lesson is written to the owning catalogue **automatically, with no permission prompt**. The gate is **structural, never a self-score**.

**First, the coverage rung** ([Evidence Discipline](evidence-discipline.md)) — *covered* (name the covering artifact; a covering tool the segment did not use → record a [bypass](project-tools.md#use-accounting) with its reason, in place of a `recurred` observation) → *extend* → *create* → *needs-evidence*, choosing the **narrowest durable owner** from `rule` · `skill` · `tool` · `pointer` · `access` · `improvement` · `drop` · `none` (already covered) · `needs-evidence`: `rule` or `tool` for a lesson from an incident (constraint, regularity, adaptation) or a reusable constraint or convention, by the [checkability rung](#checkability-rung); `tool` also for a deterministic check or operation the records show hand-rolled in at least two tasks or hand-rolled wrong; `pointer` for a file or fact the work found late — a line where an agent looks first: `docs/_index.md`, a `_index.yaml` description, a skill's description, a project role overlay (a fact never found → a todo naming it); `access` for information behind a credential, a privilege or a resource no channel or tool reaches — a todo naming the need: the resource, the information behind it, the task that hit it; a concept for the area takes it as a `proposed` channel card ([Access Channels → Sanction](access-channels.md#sanction)); the sanction stays with the developer; `improvement` for a change of the project's committed build, test or CI setup that removes a measured delay — a todo by [Run Economy → Process improvement](run-economy.md#process-improvement); the change itself waits for the developer's yes; `skill` only for knowledge that is not an artifact constraint or for the consolidated form of an accumulated rule cluster ([Knowledge Scaling](knowledge-scaling.md)), `drop` for a narrow one-off; defaulting to `skill` is forbidden. Directive, not blocking: record the reason and proceed. On *create*, open a dormant intervention-ledger entry (`lesson`/`artifact`/`owner_kind`/`problem_class`/`trigger`); a ledger failure skips the entry, never the write. On *covered*, when the lesson repeats the `problem_class` of the covering artifact's ledger entry, or a breach of that artifact, append a `recurred` observation to that entry at once, `ref` the task file — a rule's `fail` that [review](../phases/review.md) step 6 already recorded in this task, or for a tool a bypass already recorded for this check, at this rung or by a failed run ([Project Tools → Run](project-tools.md#run)), is that observation, never a second one; an artifact with an entry that fired in the segment on its sanctioned path and left a non-model by-product gets `exercised`, `ref` where the command and its result are recorded — a compliance claim without one appends nothing ([Evidence Discipline](evidence-discipline.md#consumer-3--the-intervention-ledger)). A failed append skips the observation, never the harvest. **A fact about the developer or the machine never enters this menu** — it belongs to `.dev_flow/profile/` under its consent regime, and a category recorded `declined` is **dropped**, never re-homed under another key ([Adaptation Profile](adaptation-profile.md)). This one is a refusal, not a directive-not-blocking rung. Then:

- A **reusable constraint or convention** of a `judgement` or `mixed` class → write a [rule](../phases/rule.md) in the unit form of [Knowledge Scaling](knowledge-scaling.md) — directive in the heading (within `directive_max` — split, never truncate), category selector inherited or narrowed (never widened), body within `unit_body_max` (narrative stays in the task file), provenance = the task file, digest and index updated in the same edit. Auto-written rules default to **`should`**; a `prefer` is fine. **Never auto-write a `must`** — a `must` blocks future code, so it is treated as doubtful (below).
- **Broadly-useful, non-trivial** project knowledge that passes the [skill](../phases/skill.md) non-triviality filter → write/update a skill. (Write the content as a falsifiable, evidence-scoped observation — never an absolute verdict.)
- The check of a `mechanical` or `mixed` class → **promote it to a project tool** of kind `check`. A **deterministic check or operation** hand-rolled in at least two tasks, or whose hand-rolled version gave a wrong result (name those records) → **consider promoting it to a project tool**; an agent that judges a session-workspace script worth reusing promotes it without waiting for the trigger. [Promote](project-tools.md#promote) is the one procedure for both paths: search the tool catalogue first (a covering tool → use it; a near case → `extend` it), generalize to a stated range of use, write the header contract and real acceptance samples, run them, and keep the tool as `unverified` — trust comes only from the tools step of [audit](../phases/audit.md) (samples in a clean environment, independent review, binding). The tool contract, the card, the binding and adaptation (`fix` / `extend` / `port` / `repurpose` / `retire`) live in [Project Tools](project-tools.md). Name the tool in the phase report and the commit message — `.dev_flow/` may be ignored by git. The privileged runner is never harvested ([Proportionate Caution](proportionate-caution.md#privileged-commands)).
- A `pointer` → write the line in its index or description; an `access` need or an `improvement` → the todo, named in the phase report and in the commit sign-off ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)).
- Anything narrow, trivial, or one-off → drop it.

### Checkability rung

A lesson bound for a rule — from an incident, or a reusable constraint or convention — is classified by its class (the kind of defect it guards against) on the checkability axis of [Application Enforcement](application-enforcement.md#enforcement-tiers-proportional-to-stakes--checkability--runtime), before its owner is chosen:

| Class | Decided by | Owner |
|-------|-----------|-------|
| `mechanical` | a non-model check over an artifact: a fixed pattern in a diff, a file layout, a count of surfaces | `tool` of kind `check` with the `review` phase in its `applies_to`, through [Promote](project-tools.md#promote) |
| `judgement` | only a reading: a paraphrase, whether a step carries out an obligation | `rule` |
| `mixed` | both halves | `tool` for the mechanical half; the `rule` keeps the judgement half and names the tool in its `**Check:**` line ([Knowledge Scaling](knowledge-scaling.md) unit form) |

- A `check` tool runs in the review when its `applies_to` matches the changed files and the `review` phase ([review](../phases/review.md) step 1). Its findings are an input of the review, under the severity of the rule that names the tool in a `Check:` line (`should` when no rule names it) and the materiality of [Review Convergence](review-convergence.md#finding-materiality--computed-not-judged). The tool never blocks on its own.
- A class only the runtime sees — the working directory, a command's target — leaves no artifact to check: propose a hook to the developer where the runtime has one (`interface-gate`); until it exists, `rule`. A [Phase start checklist](../SKILL.md#phase-start-checklist) item the project keeps missing — seen in a task and in an earlier one — takes the same route: a project hook proposed to the developer, installed only on their yes; no hook ships with the skill.
- Doubt between classes → `judgement`. A rule that keeps recurring at review comes back to this test in [audit](../phases/audit.md#step-6--revise-rules) Step 6.4.

**Doubtful → independent review (not suppression).** When a write is doubtful — a would-be `must`-severity rule, a **contradiction** with an existing rule/skill, or a **deletion** (a tool's included — [retire](project-tools.md#adapt)) — do not skip it and do not write it blindly: route it to an **independent clean-context review** ([Delegation for Focus](delegation.md)). It lands only if the reviewer confirms. "Doubtful" is defined by these structural triggers, not by a self-rated confidence.

**The developer's review moves to commit time, it does not disappear.** Every auto-write lands in the working tree and is surfaced in the diff under the standing commit-approval gate ([SKILL.md → Commit rules](../SKILL.md#git-workflow-integration)); mention auto-written rules/skills in the commit message and the Response Trailer so they are visible, never silent. Nothing here touches the commit gate.

This is the single home for reflection in dev-flow: [audit](../phases/audit.md) Step 3 "reflect", implement/fix **rule auto-discovery**, and [research](../phases/research.md) Step 4 persistence are all **invocations of this procedure**, not parallel paths.

### Sweep gaps

The list of [Negative sweep](../phases/specification.md#negative-sweep) questions grows from the behaviours that still reached a specification through an [upstream escalation](escalation.md) — from implement, test, review, verify or fix. The Activity entry of an escalated gap names each sweep question that asks for the behaviour the spec lacked ([Recording](escalation.md#recording)); the harvest routes each name by its row:

| The entry names | Owner |
|-----------------|-------|
| A question of the six | `none` — the shipped sweep covers it and has no ledger entry; the Activity entry is the record |
| A project rule | `none` — *covered*: append `recurred` to the rule's ledger entry, `ref` the task file, once per rule per task |
| `no sweep question` | `rule` — a new sweep question: a `judgement` class, `should`, in the category `spec-sweep` (`applies_to: { phases: [spec] }`, created with its first rule — [Rule Categories](../phases/rule.md#rule-categories)); the directive is the question for that kind of input, written as one imperative line ("Say what happens when …"), the example the spec section that lacked the answer and the section that now gives it; its ledger entry opens on create (`problem_class: spec-sweep:<kind of input>`, `trigger`: the next task that runs the specification phase) |

Any other escalation to a specification — an ambiguity, a wrong value, a change of a published contract — carries no sweep name and goes through the owner menu above like any lesson.

## The Response Trailer — response-granularity carrier

Between checkpoints, an individual response can still carry salience and signal a pending transition with a compact **trailer** — a single labelled line, factual fields only:

```
⟦checkpoint⟧ segment · type · decisions · artifacts · {s:…} · transition?
```

| Field | Meaning |
|-------|---------|
| `segment` | The segment label this response belongs to |
| `type` | `decision` / `exploration` / `refinement` / `status` |
| `decisions` | DEC ids settled or opened (if any) |
| `artifacts` | Files touched (if any) |
| salience | the entry's `{s:…}` value (`pin` / `normal` / `noise`) |
| `transition?` | does this response close a segment (fire a checkpoint)? |

**Factual fields only.** A subjective numeric self-score — confidence %, quality N/10 — is **not permitted**. The durable copy is the **task-file entry**; a *visible* footer is optional and reserved for semantically complex responses (it need not appear on every turn).

**Pre-action sibling.** The same fact-only, pointer-only idiom runs *forward* as well as back: [Application Enforcement](application-enforcement.md)'s **Pre-Action Marker** re-surfaces the applicable rules/skills/tools (+ the skills' freshness) *before* an action burst, where this trailer reports *after* one.

## Context pressure — a tiered, proxy-driven trigger

A checkpoint can also fire because the context is filling up — but assessed by **proxy, never by introspection, and never by riding the harness's own compaction**:

- **Signal** — `window_fill` if (and only if) the runtime exposes it; otherwise derive pressure from **structural proxies**: turn count, task-file size, repetition signals, error rate.
- **Tiers** — the *pressure level* (left) names the input severity; `AssessContextPressure` returns the matching *response action* (in parentheses):
  - **ok** — nothing to do. *(action: `ok`)*
  - **moderate** — run a checkpoint now (distill + pin the summary, demote raw), stay lean. *(action: `checkpoint`)*
  - **high + degradation** — ensure the task files are fully checkpointed and **always-resumable**, then **recommend a graceful session handoff** to the developer: a deterministic restart-from-files, not a lossy auto-compact. The recommendation is a command, not a wish — [`/dev-flow checkpoint`](../phases/status.md#checkpoint--fix-the-task-for-a-session-boundary) fixes the task and writes the handoff record, and the fresh session opens with [`/dev-flow resume`](../phases/status.md#resume--re-enter-a-task-in-a-fresh-session). *(action: `recommend-handoff`)*

**Two senses of "checkpoint".** The `moderate` tier's action is *this* procedure — the in-context Transition Checkpoint above. The `checkpoint` **command** is the developer-invoked form: it runs the same distillation and then does what a structural boundary does not — closes documentation drift, satisfies the handoff readiness set, and records where to restart. One is how the agent stays lean; the other is how a session ends.

**Honest boundary.** The agent cannot prevent a harness auto-compact, nor reliably read the window fill. So `recommend-handoff` is exactly that — a recommendation to the developer, backed by the standing duty to keep task files always-resumable — not an action the agent can force. It must **never** trigger an automatic compaction, never run the `checkpoint` command on the developer's behalf, and never decide the tier from self-assessment.

## Relation to working memory

Experience Capture is what fires the L1→L2 movements of [session working memory](cache.md#session-working-memory-l1):

- **At a checkpoint** — *promote to durable*: a settled decision, a confirmed parameter, a harvested lesson moves into `.dev_flow/`; the rest stays L1 scratch.
- **After a compaction / on a subtask switch** — *re-attention*: re-read the working-memory area to rebuild focus before continuing (the [status read protocol](../phases/status.md#read-protocol) opens with this).

## Where it is wired

| Phase / moment | How Experience Capture applies |
|----------------|-------------------------------|
| implement / fix / verify | Checkpoint at phase and subtask boundaries; trailer on complex responses; context-pressure assessed as the session grows |
| [`checkpoint` command](../phases/status.md#checkpoint--fix-the-task-for-a-session-boundary) | Its first movement **is** a Transition Checkpoint invocation, run out of band because the developer's invocation is the transition; harvest still precedes demotion |
| [audit](../phases/audit.md) Step 3 (reflect) | The reflect step **is** a checkpoint invocation — harvest lessons from a closing task and auto-apply them as rules/skills/tools through the structural gate |
| [research](../phases/research.md) Step 4 | Persisting durable spike findings is the same harvest-and-auto-apply move |
| [review](../phases/review.md) step 6 | A project rule the round's report marks `fail` appends `recurred` to its ledger entry, once per rule per task |
| [Task close](../phases/status.md#on-task-completion) | When no checkpoint ran over the task's last segment, one runs before the sign-off of the task's last commit is asked, so its writes are in the diff the developer approves |
| A delegate's report | Its `Friction:` items are Delay watch signals of the initiator's task ([A delegate's run](delivery-focus.md#a-delegates-run)) |
| [Upstream Escalation](escalation.md#recording) of a gap in a specification | Its Activity entry names each sweep question that asks for the behaviour; the next checkpoint routes it by [Sweep gaps](#sweep-gaps) |
| Specialist end-of-burst | A project specialist accrues role-local heuristics **per call** (it self-writes its own memory, so the next call warm-starts on them); this per-call append is **ungated** (only the skill promotion below passes a gate), so — applying "trust structure, not self-assessment" to memory *content* — each heuristic is written as a *falsifiable, evidence-scoped observation*, never an absolute verdict (an over-stated one poisons later warm-starts). At **end-of-burst** this procedure harvests the burst's heuristics and *auto-applies* the broadly-useful ones as skills through the structural gate (hybrid store — role-local memory + skills) |
| Optional `/dream` | If a reflection/`dream` skill is installed, delegate the distillation step to it for a deeper retrospective; otherwise distill inline. Experience Capture does not depend on `/dream`. |

## A note on these guidelines

This is a **projection, not a formal protocol** — heuristics the agent applies with judgment, the same footing as [Delegation for Focus](delegation.md) and [Interview Mode](interview-mode.md). Don't encode rigid MUST/NEVER machinery beyond the load-bearing ones — *factual-only trailers* and *harvest-before-demote* — and keep the rest as prose.

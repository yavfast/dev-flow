# Experience Capture — Reflection & Salience at Transitions

Cross-cutting sub-procedure for every phase. Like [Delegation for Focus](delegation.md),
it is **not** a pipeline stage — it runs *inside* whatever phase is executing, firing at
the boundaries between stretches of work. Where delegation protects *attention* by moving
noise out of the context, Experience Capture protects *memory*: it distills what a stretch
of work produced — the decision, the lesson, the one line worth keeping — **before** the
raw turns that produced it are compacted away.

## Why this exists

Compaction is where context is lost, and dev-flow archives by **age** — the oldest turns
go first, whether or not they still matter. A long session pays for this twice: the raw
back-and-forth of solving something is bulky and mostly noise once solved, yet the *lesson*
inside it is exactly what you want to survive. Left alone, the summary keeps the noise and
drops the signal.

Two design commitments shape the cure, both learned the hard way (and confirmed by prior
art — Generative Agents, Reflexion, Voyager, MemGPT, MemoryBank):

- **Store the distillation, not the transcript.** What survives is a short summary plus any
  durable lesson — never the raw turns, which are referenced by path if kept at all.
- **Trust structure, not self-assessment.** An agent's numeric self-rating ("importance
  8/10", "confidence 90%") is unreliable. So reflection fires on **deterministic structural
  events** dev-flow already logs, and salience is recorded with the **discrete markers** of
  [Salience Markers](../phases/status.md#salience-markers) — never a self-scored number.

The north star behind all of it: keep durable task state **complete in L2** (`.dev_flow/`)
so the transcript becomes disposable and a fresh session can restart from files
deterministically — which beats riding a lossy context-summary (see the
[memory tiers](cache.md#memory--data-tiers-l0l1l2)).

## The unit: a segment

A **segment** is a coherent stretch of work on one topic or task, bounded by two
transitions — e.g. "spec-SP_SAL" from opening the spec to finishing it. It has a short
label, an owning task, and a lifecycle of exactly `open → closed` (closed once, by the
transition that ends it). The label is what a response's trailer names; the checkpoint at
its closing boundary is what distills it.

## Transition Checkpoint — the primary cadence

At a transition, run a **checkpoint** over the segment that is closing. This formalizes
behavior good agents already do ad hoc; the value is doing it *reliably, at known
boundaries*.

**Detecting a transition.** Anchor on **structural** signals — `phase-boundary`,
`task-switch`, `subtask-done` — which are deterministic events dev-flow already records.
**Semantic** signals (a visible topic-shift, an explicit marker you raise) are a
best-effort secondary: useful for an intra-phase checkpoint, but **never the sole basis
when a structural signal is also available**, and self-detected topic boundaries alone are
never a valid trigger (that is the self-assessment trap again).

**What a checkpoint does**, in order:

1. **Distill** the closing segment — a noisy read, so delegate it (see
   [Delegation for Focus](delegation.md)) and keep only the conclusion.
2. **Record a segment summary** as a Shared Activity Log entry tagged `{s:pin}` — a few
   lines of "what happened / what was decided", referencing any raw material by path,
   never inlining it (the no-large-blobs rule).
3. **Demote the raw turns** of the segment — re-grade them `{s:noise}`, or
   `{s:superseded→<summary>}` when the summary replaces them.
4. **Propose durable lessons** (next section) — proposed, never applied.
5. **Close the segment**, and **promote** the durable part of working memory to L2 (see
   [Relation to working memory](#relation-to-working-memory)).

The pinned summary now survives a dev-flow compaction while the raw turns are evicted first
— the signal kept, the noise shed.

## Harvest before demote (ordering invariant)

When a task **closes**, its salience markers go inert (a `pin` is task-scoped — see
[Salience Markers](../phases/status.md#salience-markers)). The checkpoint that harvests the
closing task's lesson MUST run **before** that demotion — reflect first, demote second — so
a durable lesson is never lost in the moment its `pin` expires.

## Proposing experience — propose, never apply

A harvested lesson is routed to the owning knowledge phase as a **proposal**, never written
into the catalogues by this procedure:

- A **reusable constraint or convention** → propose a [rule](../phases/rule.md) (default
  severity `should` unless clearly `must`/`prefer`).
- **Broadly-useful, non-trivial** project knowledge that passes the
  [skill](../phases/skill.md) non-triviality filter → propose a skill.
- Anything narrow, trivial, or one-off → drop it.

A proposal starts `proposed` and becomes `confirmed` only through the rule/skill phase's
own gate (developer accepts) — Experience Capture never auto-writes. This is the single
home for reflection in dev-flow: [audit](../phases/audit.md) Step 3 "reflect",
implement/fix **rule auto-discovery**, and [research](../phases/research.md) Step 4
persistence are all **invocations of this procedure**, not parallel paths.

## The Response Trailer — response-granularity carrier

Between checkpoints, an individual response can still carry salience and signal a pending
transition with a compact **trailer** — a single labelled line, factual fields only:

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

**Factual fields only.** A subjective numeric self-score — confidence %, quality N/10 — is
**not permitted**; it is the unreliable signal the whole design avoids. The durable copy is
the **task-file entry**; a *visible* footer is optional and reserved for semantically
complex responses (it need not appear on every turn).

## Context pressure — a tiered, proxy-driven trigger

A checkpoint can also fire because the context is filling up — but assessed by **proxy,
never by introspection, and never by riding the harness's own compaction**:

- **Signal** — `window_fill` if (and only if) the runtime exposes it; otherwise derive
  pressure from **structural proxies**: turn count, task-file size, repetition signals,
  error rate. The trigger is portable across runtimes precisely because it does not depend
  on a fill number the runtime may hide.
- **Tiers:**
  - **ok** — nothing to do.
  - **moderate** — run a checkpoint now (distill + pin the summary, demote raw), stay lean.
  - **high + degradation** — ensure the task files are fully checkpointed and
    **always-resumable**, then **recommend a graceful session handoff** to the developer:
    a deterministic restart-from-files via the [status phase](../phases/status.md), not a
    lossy auto-compact.

**Honest boundary.** The agent cannot prevent a harness auto-compact, nor reliably read the
window fill. So `recommend-handoff` is exactly that — a recommendation to the developer,
backed by the standing duty to keep task files always-resumable — not an action the agent
can force. It must **never** trigger an automatic compaction, and must never decide the tier
from self-assessment.

## Relation to working memory

Experience Capture is what fires the L1→L2 movements of
[session working memory](cache.md#session-working-memory-l1):

- **At a checkpoint** — *promote to durable*: a settled decision, a confirmed parameter, a
  harvested lesson moves into `.dev_flow/`; the rest stays L1 scratch.
- **After a compaction / on a subtask switch** — *re-attention*: re-read the working-memory
  area to rebuild focus before continuing (the [status read
  protocol](../phases/status.md#read-protocol) opens with this).

## Where it runs (touchpoints)

| Phase / moment | How Experience Capture applies |
|----------------|-------------------------------|
| implement / fix / verify | Checkpoint at phase and subtask boundaries; trailer on complex responses; context-pressure assessed as the session grows |
| [audit](../phases/audit.md) Step 3 (reflect) | The reflect step **is** a checkpoint invocation — harvest lessons from a closing task into proposed rules/skills |
| [research](../phases/research.md) Step 4 | Persisting durable spike findings is the same propose-experience move |
| Specialist end-of-burst | A project specialist's distilled heuristics are harvested via this procedure (hybrid store — role-local memory + skill proposals) |
| Optional `/dream` | If a reflection/`dream` skill is installed, delegate the distillation step to it for a deeper retrospective; otherwise distill inline. Experience Capture does not depend on `/dream`. |

## A note on these guidelines

This is a **projection, not a formal protocol** — heuristics the agent applies with
judgment, the same footing as [Delegation for Focus](delegation.md) and
[Interview Mode](interview-mode.md). A spurious checkpoint costs one cheap summary, not
corruption; a missed semantic shift is caught at the next structural boundary; an
over-eager cadence is flagged by audit. Don't encode rigid MUST/NEVER machinery beyond the
two that are load-bearing — *factual-only trailers* and *harvest-before-demote* — and keep
the rest as prose.

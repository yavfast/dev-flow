# Phase: Research — Time-Boxed Investigation (Spike)

## Purpose

Close a knowledge gap **before** committing to a design. Research is the phase for
situations where a concept, specification, or plan cannot be confidently authored
because data or context is missing: an unfamiliar domain, an unverified library
capability, an unknown solution space, or an open Design Decision waiting on facts.

The output is a **spike** — a time-boxed, throwaway investigation recorded in
`docs/{name}.spike.md` — plus durable findings persisted to `.dev_flow/skills/`.
Research produces *knowledge and candidate options*, never designs: the concept
still decides, the interview still chooses.

Research is **not** a numbered pipeline stage. It is an on-demand service phase,
invoked before or during Concept/Spec/Plan whenever proceeding would mean
building on a guess.

## Command

```
/dev-flow research <question or topic>

# alias:
/dev-flow spike <question or topic>
```

### Examples

```
/dev-flow research can library X handle streaming uploads over 2GB?
/dev-flow research порівняй підходи до rate limiting: token bucket vs sliding window
/dev-flow research what sync strategies exist for offline-first mobile clients?
/dev-flow research closing C_SYNC_DEC_02 — measure baseline latency of approach A
```

## Role Responsible

Investigation is delegated to **Researcher**:
[roles/researcher.ai.md](../roles/researcher.ai.md)

The main agent frames the spike, receives the conclusion, persists durable
knowledge, and updates decision records. The researcher subagent does the noisy
exploration — see [Delegation for Focus](../references/delegation.md).

## Entry Points

| Triggered from | Signal |
|----------------|--------|
| **Explicit call** | User runs `/dev-flow research` / `spike` |
| **[Do router](do.md)** | Request contains "дослідити", "research", "spike", "compare approaches", "не знаю, який підхід", or the answer requires external sources / experiments |
| **[Concept phase](concept.md)** | A Pre-Concept Checklist answer rests on an unverified assumption — research before authoring on a guess |
| **[Interview Mode](../references/interview-mode.md)** | The solution space is unknown (no options to interview on yet), or an **open** Design Decision's resolution trigger names missing information ("resolve once load-test numbers exist") |
| **[Ask phase](ask.md)** | The question cannot be answered from the codebase and docs alone — ask stays read-only and suggests escalating here |
| **[Audit phase](audit.md)** | An open decision's trigger has expired and the missing information is researchable |

## When NOT to Use

- The answer is already in `.dev_flow/skills/` — the skill check below catches this.
- The answer is derivable from the codebase or docs — that is [ask](ask.md), not research.
- The options are already known and only a choice is needed — that is an
  [interview](../references/interview-mode.md), not a spike.
- The "research" is actually the main task (e.g. a performance investigation that
  *is* the deliverable) — run it as a normal task, not a spike.

## Context Loading

**Skill check (gate).** MUST read `.dev_flow/skills/_index.yaml` and load skills for the
research topic BEFORE investigating — an existing skill may already answer the question
or narrow it. See [skill phase](skill.md).

**Cache check (gate).** Read `.dev_flow/cache/_index.yaml` (if present) — the needed
resource (design export, downloaded document, data sample) may already be fetched;
reuse it instead of spending limited access. See [cache phase](cache.md).

**Glossary check.** Load `docs/_glossary.md` (if present) so questions and findings use
canonical domain terms. See [Glossary](../references/glossary.md).

## Cost Gate

Research can sink unbounded time and tokens. Before starting, frame it explicitly
(same discipline as the [fix phase's diagnosis gate](fix.md)):

1. **Questions** — 1–3 specific questions the spike must answer. Not "learn about X"
   but "can X do Y under constraint Z?".
2. **Scope** — which sources are in bounds: `codebase` (read existing code/docs),
   `external` (web, package registries, official docs), `prototype` (throwaway code
   in a scratch area). Combine as needed.
3. **Time-box** — the budget ("2 hours", "1 session", "10 sources max"). When the
   box is spent, the spike concludes with whatever it has — `concluded` or
   `inconclusive`, honestly.

When the expected cost is non-trivial, present the fork instead of silently sinking effort:

```
A) Run the spike — {questions}, scope: {scope}, time-box: {box}   (Recommended: {why})
B) Proceed without research — record the assumption as an open Design Decision
   with a resolution trigger
C) You provide the missing information directly (doc, measurement, prior art)
```

## Procedure

### Step 1: Frame the spike

Apply the cost gate above. Create `docs/{name}.spike.md` from the
[spike template](../templates/spike.md): questions, context, scope, time-box,
target concept (or "to be created"), or the open `DEC_NN` it serves.

### Step 2: Investigate (delegated)

Spawn the [Researcher](../roles/researcher.ai.md) subagent with the spike file as
its brief. The researcher:
- exhausts cheap sources first (codebase evidence → existing docs → external sources →
  prototype only if scoped),
- logs each exploration entry in the spike file (what was tried, findings, open questions),
- stages fetched artifacts (downloads, design exports) under the project workspace
  `/tmp/{project-slug}/downloads/` and lists their paths in the report
  (see [cache phase](cache.md) — helper subagents never write `.dev_flow/`;
  the agent running this phase persists),
- fills the Alternatives Considered table,
- concludes with a verdict and returns **the conclusion, not the dump**.

For trivially small lookups, the main agent may run Step 2 inline — but any
investigation that would flood the context (multiple sources, prototype runs)
goes through the subagent.

### Step 3: Conclude the spike

Set the spike status: `concluded` (questions answered), `inconclusive` (time-box
spent, reasons recorded), or `abandoned`. The Conclusion section must state:
verdict, key constraints discovered, recommendations for the concept, and
artifacts to discard.

### Step 4: Persist durable knowledge & artifacts

Findings that pass the [skill phase's non-triviality filter](skill.md) are saved
to `.dev_flow/skills/` **by the agent running this phase** — main agent or
task-delegated executor; its helper subagents do not write `.dev_flow/`.
This is the owner of the rule "web research → consolidated skill": research is
where that consolidation happens, so the next task starts ahead.

The same split applies to **artifacts**: staged files the spike's conclusions rest
on — or that were expensive to fetch (rate-limited Figma access, large downloads) —
are promoted from the workspace to `.dev_flow/cache/` with index entries (see
[cache phase](cache.md)); the rest of the staging area is left to die with `/tmp`.
Record promoted files under the spike's "Artifacts to keep".

### Step 5: Close what the spike unblocks

- **Open Design Decision** — if the spike was the resolution trigger of a `DEC_NN`,
  update the record: research informs, the developer still decides. Present the
  findings as a short interview (options + recommendation) and flip the record to
  `resolved` with the rationale; or extend the trigger if genuinely still open.
- **Pre-Concept Checklist** — restate the previously-unverified answer with the
  spike's evidence; the concept references the spike in its `Spike:` metadata field.

### Step 6: Hand off

Report: verdict per question, recommended next phase (`concept` ready / more
research needed / not viable — drop), and what was persisted where. Update the
task file per the standard [rules for all phases](../SKILL.md#rules-for-all-phases).

## Spike Rules

- A spike is **throwaway research**, not a deliverable. It does not pass through
  validation gates, and no gate may require its *content* — only its *conclusion*.
- Spike conclusions **feed** the concept/spec/plan — they never replace them, and a
  spike never makes the design decision itself.
- The time-box is binding. A spike that overruns its box concludes `inconclusive`
  with reasons — it does not silently extend.
- Prototype code lives in a scratch area, is listed under "Artifacts to discard",
  and must NOT become production code.
- A spike answers its framed questions. New questions discovered mid-flight are
  recorded as open questions — re-frame (a new cost gate) before chasing them.

## Relation to Other Phases

| Phase | Relation |
|-------|----------|
| [concept](concept.md) | Research runs *before* authoring when the checklist rests on guesses; the concept links the spike via its `Spike:` field |
| [Interview Mode](../references/interview-mode.md) | Spike *discovers* options; interview *chooses*. Research is the sanctioned closure path for open decisions waiting on facts |
| [ask](ask.md) | Ask answers from what exists (read-only); research goes beyond it (external sources, experiments) and persists what it learns |
| [skill](skill.md) | Research is the producing side of "external research → consolidated skill" |
| [cache](cache.md) | Checked before external fetches; Step 4 promotes durable artifacts (downloads, design exports) there from the workspace |
| [subtask](subtask.md) | A research run is a natural subtask — delegated, scoped, reported back |
| [audit](audit.md) | Audit flags expired open-decision triggers and stale `in-progress` spikes, and proposes research to close them |

## Anti-Patterns

- Researching what an existing skill or the codebase already answers
- "Learn about X" spikes with no specific questions and no time-box
- Letting a spike grow into the implementation (prototype quietly becoming production code)
- Making the design decision inside the spike instead of bringing options back to the interview
- Burying findings in the spike file without persisting the durable part to `.dev_flow/skills/`
- An `inconclusive` verdict with no record of *what* is still missing and what would close it

## Language Policy

Frame the spike and report findings in the **same language** the user used.
Spike file section headers stay as the template defines them.

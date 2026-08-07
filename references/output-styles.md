# Output Styles — Documentation Register vs Conversation Register

Cross-cutting sub-procedure of **every phase that writes a documentation file or answers the developer**. Not a pipeline stage, no command. **Advisory:** loading the project file is a gate, applying a profile is judgment at the moment of writing — it adds no gate criterion. Text *mechanics* belong to [Documentation Formatting](formatting.md); this reference owns the *register* — language, semantic complexity, and how a pointer to another document is carried.

## Why this exists

dev-flow writes to two surfaces with different readers. **Documentation** is read mostly by an agent (a later session, a subagent, the clean-context reviewer) and skimmed by the developer — a cross-reference there **replaces** a restatement, which is what keeps one canonical statement per concern. **Chat** is read by the person deciding right now; the link is not followable (in an Interview Mode dialog it often does not render as a link at all), so a reference there **needs** a restatement. Same asymmetry, opposite conclusions: in a document a link saves words, in chat it costs them. And the price of a half-understood sentence differs — a document gets re-read, a chat message gets acted on.

## The surfaces

| Surface | Reader | What is written there |
|---------|--------|----------------------|
| `documentation` | Agent first, skimming developer second | Concept, spec, plan, spike, epic, task file, index, rule file, skill file, `_glossary.md`, `_framework.md` |
| `chat` | The developer, deciding from this message | Phase report, Interview Mode question, commit-approval request, explanation, `ask` answer, relayed subtask report |

Commit messages belong to neither — they keep their own conventions ([Git Workflow](../SKILL.md#git-workflow-integration)).

## Shipped default — `documentation`

- Cross-reference instead of restating: point at the traceable ID or anchor, do not repeat someone else's wording.
- Let structure carry meaning — headings, tables, stable anchors.
- Write in the project's documentation language; never mix languages inside one document body.
- English text targets the ASD-STE100 register (below).
- Mechanics per [Documentation Formatting](formatting.md).

## Shipped default — `chat`

- Answer in the language the developer writes in, unless the project profile says otherwise.
- Keep semantic complexity low: short sentences, one idea per sentence, no nested subordinate clauses, no unexplained jargon. A half-understood sentence here produces a wrong decision, not a style complaint.
- **Every link, path, or traceable ID carries a short description** — what is behind it and why it is mentioned. The description *accompanies* the link, it does not replace it: where the jump works, it keeps working.
- Say what is needed to decide. Detail stays in the documents; chat carries what changed and what the developer must choose.
- English text targets the ASD-STE100 register (below).

```md
<!-- WRONG in chat — the sentence only works if the jump works -->
Blocked by SP_RL_02_03 — see docs/rate_limiter.sp.md.

<!-- RIGHT in chat — readable with the jump dead -->
Blocked by SP_RL_02_03 (docs/rate_limiter.sp.md) — the token-bucket refill
contract: it caps a burst at 100, and the new endpoint asks for 500.
```

**[Interview Mode](interview-mode.md) is the sharpest case.** In an options dialog the question text *is* the whole context: markup may not render, the jump does not work, and the answer is irreversible by the definition of a fork. Self-containment there is the condition for an informed choice, not a nicety.

## English register — ASD-STE100

When the text is in English, the target register on both surfaces is **ASD-STE100** (Simplified Technical English). The skill names the standard as an orientation: it ships neither its rule set nor its dictionary, and claims no compliance with it. A project that wants concrete register rules writes them into its own profile (next section).

## The project file — `.dev_flow/output_styles.md`

**Load gate.** Read it at the start of every phase, alongside `.dev_flow/rules/` and `.dev_flow/skills/`. Absent → the shipped defaults apply and the gate is a no-op.

One file holds every profile of the project. Shape mirrors `.dev_flow/rules/*.md`:

```md
## Profile: {name}

**Applies to:** {documentation|chat} [ · phases: {a}, {b} ] [ · when: {condition} ]

- {directive}
- {directive}
```

**Selection.** The shipped default for the surface is the base layer; matching project profiles overlay it in ascending order of narrowness (surface-only first, phase- or condition-scoped last). Equal narrowness → the project profile wins. A profile that does not parse (no `Applies to:`, unknown surface, empty directives) is skipped and mentioned in the phase report; the rest of the file still works.

**Adaptive maintenance.** Create the file lazily — at the first preference the developer actually states, not up front. Write a directive when they state or correct one ("shorter", "docs in English", "no bare links"); when you are merely *guessing* at a taste, ask first, then write. A [Transition Checkpoint](experience-capture.md) that harvests a durable style lesson files it here, not into rules or skills. A preference that is really a code convention routes to [`/dev-flow rule`](../phases/rule.md) instead.

## Restraints

- **Style never removes content.** A blocker, an open decision, an intent divergence, a stated uncertainty, or an `unobserved` with its reason is written out even when brevity suffers. Simplification is about delivery, never about facts.
- **Fact-only carriers are not restyled** — the Response Trailer ([Experience Capture](experience-capture.md)), the Pre-Action Marker ([Application Enforcement](application-enforcement.md)), and the report ceiling ([Evidence Discipline](evidence-discipline.md)) stay as they are; the profile governs the prose around them.
- **Identifiers are not translated** — traceable IDs, paths, file names, canonical glossary terms, and command names stay as they are whatever the language of the sentence.
- **Read the language off the developer's message**, not off a locale, a name, or the previous project.
- **Documentation language and chat language are independent** — English docs with a Ukrainian conversation is a valid project profile, not a contradiction.
- **A new or changed profile is not a reason to rewrite existing documents** — the same no-wholesale-reformatting restraint as [Documentation Formatting](formatting.md).
- **Not a home for code conventions** — those stay in `.dev_flow/rules/`, with severity; a style profile has none and blocks nothing.

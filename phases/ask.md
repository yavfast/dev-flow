# Phase: Ask — Question About Code or Feasibility

## Purpose

Answer questions about the project's codebase, architecture, or feasibility of new
functionality — **without making any changes** to code, docs, or active context.

This is a read-only, advisory phase. It explores the codebase, analyzes existing
concepts/specs/plans, and provides a well-grounded answer.

## Command

```
/dev-flow ask <question>
```

### Examples

```
/dev-flow ask як працює маршрутизація в agent_loop?
/dev-flow ask can we add WebSocket support to the gateway without breaking the REST API?
/dev-flow ask which modules would be affected if we replace litellm with a custom adapter?
/dev-flow ask де визначені контракти для ToolSandbox?
/dev-flow ask is it feasible to add multi-tenant support given the current architecture?
```

## Role Responsible

This command is handled by **Advisor**:
[roles/advisor.ai.md](../roles/advisor.ai.md)

## Constraints

- **No file writes.** Do not create, edit, or delete any files.
- **No active context updates.** Do not touch `.dev_flow/active_context.md`.
- **No git operations.** No commits, branches, or stashes.
- **No code generation.** Do not produce implementation code (pseudocode for illustration is OK).

## Procedure

### Step 1: Classify the question

| Question type | Indicators | Approach |
|---------------|-----------|----------|
| **Code understanding** | "how does X work", "what does Y do", "де визначено Z" | Read source, trace call chains, explain |
| **Feasibility analysis** | "can we", "is it possible", "чи можливо", "what would it take" | Analyze architecture, identify constraints, assess effort |
| **Impact assessment** | "what would break", "which modules affected", "що зміниться" | Trace dependencies, list affected files and contracts |
| **Architecture question** | "why is X designed this way", "чому так", "what pattern" | Read code + docs, explain design decisions |
| **Where to find** | "where is", "де знаходиться", "which file" | Search codebase, return file paths and relevant lines |

### Step 2: Gather context

1. Read relevant source files, concepts (`*.concept.md`), specs (`*.sp.md`), and plans (`*.plan.md`).
2. If the project has `.dev_flow/active_context.md` — read it for current work context (but do not modify).
3. **Skill check:** Read `.dev_flow/skills/_index.yaml` and load skills relevant to the
   question topic — they may contain domain knowledge, known pitfalls, or architectural
   context that informs the answer. See [skill phase](skill.md).
4. **Rule check:** Read `.dev_flow/rules/_index.yaml` (if exists) and load rules relevant
   to the question area — they document project conventions and constraints. See [rule phase](rule.md).
5. Use Grep/Glob to trace references, imports, and usage patterns.
6. For feasibility questions — also check existing tests and integration points.

### Step 3: Formulate the answer

Structure the answer based on question type:

**For code understanding:**
- What the code does (high-level)
- Key files and entry points (with paths and line references)
- Data flow or call chain

**For feasibility analysis:**
- Short verdict: feasible / feasible with caveats / not feasible
- What exists today that supports this
- What would need to change (list affected layers: concept / spec / plan / code)
- Estimated scope: small (1 spec section), medium (1 concept), large (multiple concepts)
- Risks or open questions

**For impact assessment:**
- List of affected files and modules
- Which concepts/specs/plans would need updating
- Breaking vs non-breaking change assessment

**For architecture questions:**
- The design decision and its rationale
- Relevant code references
- Trade-offs that were made

**For where-to-find:**
- File path(s) with line numbers
- Brief description of what's there

### Step 4: Suggest next steps (optional)

If the question naturally leads to action, suggest which dev-flow command to run next:

> "If you want to proceed, run `/dev-flow concept` to define the new capability,
> or `/dev-flow do <description>` to let the orchestrator handle routing."

This is a suggestion only — do not execute anything.

## Language Policy

Respond in the **same language** the user used in their question.

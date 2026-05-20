# dev-flow

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that enforces **concept-driven development** — a structured pipeline where every code change traces back to a concept and specification, preventing architectural drift and ensuring living documentation.

## Why dev-flow?

Most projects suffer from a common pattern: documentation is written once and forgotten, specs diverge from code, and architectural decisions get lost in commit history. dev-flow solves this by making documentation a **first-class artifact** in the development pipeline, not an afterthought.

- **Ideas before code** — concepts and specs are written before implementation, catching design flaws early
- **End-to-end traceability** — every code section links back to its concept/spec via immutable IDs
- **Living documentation** — change propagation keeps docs in sync with code
- **Validation gates** — prevent incomplete specs from becoming buggy code
- **Unbiased code review** — pre-commit review by a clean-context subagent (no implementer blind spots)
- **Session continuity** — pick up where you left off across conversations
- **Works for new and existing projects** — greenfield pipeline or reverse-engineer docs from existing code

## The Pipeline

```
Concept → [Gate] → Spec → [Gate] → Plan → [Gate] → Code → [Gate] → Test → [Gate] → Review → [Gate] → Verify → Commit → Propagate
```

Each transition has a **validation gate** that checks completeness before advancing:

| Phase | Command | What it does |
|-------|---------|-------------|
| Onboard | `/dev-flow onboard` | Reverse-engineer docs from existing code (run once) |
| Concept | `/dev-flow concept` | Define the idea — philosophy, domain model, mechanisms |
| Spec | `/dev-flow spec` | Define data structures, contracts, validation rules |
| Plan | `/dev-flow plan` | Break spec into actionable implementation phases |
| Implement | `/dev-flow implement` | Write code following the plan |
| Test | `/dev-flow test` | Run functional tests (unit + mock) |
| Review | `/dev-flow review` | Pre-commit code review by clean-context subagent |
| Verify | `/dev-flow verify` | Regression, integration, and live testing |
| Propagate | `/dev-flow propagate` | Update docs when code changes |

Additional commands:

| Command | Purpose |
|---------|---------|
| `/dev-flow fix <problem>` | Investigate and fix a bug |
| `/dev-flow ask <question>` | Read-only Q&A — no file changes |
| `/dev-flow rule <request>` | Manage project coding rules |
| `/dev-flow skill <request>` | Manage project technology knowledge |
| `/dev-flow status` | Show current state, resume previous session |
| `/dev-flow <anything>` | Freeform — auto-routes to the right phase |

## Installation

Copy the skill into your Claude Code skills directory:

```bash
# Clone the repository
git clone https://github.com/yavfast/dev-flow.git

# Copy to Claude Code skills directory
cp -r dev-flow ~/.claude/skills/dev-flow
```

Or add as a git submodule in your project:

```bash
git submodule add https://github.com/yavfast/dev-flow.git .claude/skills/dev-flow
```

## Quick Start

### New project (greenfield)

```
> /dev-flow concept rate limiter for API calls

Creating concept: docs/rate_limiter.concept.md
- Philosophy: per-agent token bucket, delay instead of reject
- Domain model: Agent → RateBucket { capacity, tokens, refill_rate }
- Data flows: Agent calls API → Limiter checks bucket → proceed / wait

Gate check: ✓ No conflicts. Scope clear. Ready for spec.
```

```
> /dev-flow spec

Creating specification: docs/rate_limiter.sp.md
- Data structures: RateBucket (5 fields, all typed with constraints)
- Contracts: AcquireToken (input/output/errors/pseudocode)
- Validation rules, state transitions, verification criteria

Gate check: ✓ All fields typed. Error cases specified. Proceed to plan.
```

```
> /dev-flow plan

Creating plan: docs/rate_limiter.plan.md
- Technology: Python 3.12, token bucket algorithm
- Phase 1: RateBucket data model
- Phase 2: Integration with LLMRouter

Gate check: ✓ All spec sections covered. Dependencies stated.
```

```
> /dev-flow implement

Implementing Phase 1...
- Created engine/rate_limiter.py
- Added # [SP_RLM_01_01] traceable ID comments
- Following SOLID principles (DIP: injectable storage)
Updated plan: Phase 1 [TODO] → [DONE]
```

### Existing project (takeover)

```
> /dev-flow onboard

Analyzing project structure...
- Layer 0: utils/, helpers/ (no internal dependencies)
- Layer 1: models/ (depends on utils)
- Layer 2: services/ (depends on models)
- Layer 3: api/ (depends on services)

Generating documentation...
- Created docs/utils.concept.md
- Created docs/models.concept.md
- Created docs/services.concept.md
- Extracted 12 coding rules to .dev_flow/rules/
```

### Day-to-day workflow

```
> /dev-flow fix login fails when email has uppercase letters

Investigating...
- Found: email comparison in auth_service.py:42 is case-sensitive
- Spec SP_AUTH_02_01 says: "email matching is case-insensitive"
- Fix: normalize email to lowercase before comparison
- Tests pass. Review pass. Verify pass.
Ready to commit?
```

```
> /dev-flow add dark mode toggle to settings page

Routing to: concept → spec → plan → implement
(walks you through the full pipeline)
```

```
> /dev-flow ask can we replace Redis with SQLite for rate limiting?

Read-only analysis:
- RateLimiter depends on BucketStorage interface (DIP)
- Current impl: RedisBucketStorage
- SQLite would work for single-instance deployments
- Impact: 1 new class, 0 spec changes, 0 concept changes
- Risk: no distributed locking — won't work with multiple instances
```

## How It Works

### Traceable IDs

Every document section gets an immutable identifier:

| Document | Format | Example |
|----------|--------|---------|
| Concept | `C_XXX_NN_NN` | `C_RLM_01_01` |
| Specification | `SP_XXX_NN_NN` | `SP_RLM_02_01` |
| Plan | `PL_XXX` | `PL_RLM` |

These IDs appear as comments in code, linking implementation back to design:

```python
# [SP_RLM_01_01] RateBucket — token bucket data model
@dataclass
class RateBucket:
    agent_id: str
    capacity: int = 10
    ...

# [C_RLM_02_01] RateLimiter — per-agent token bucket management
# [SP_RLM_02_01] AcquireToken contract
class RateLimiter:
    ...
```

### Validation Gates

Each gate checks specific criteria before allowing advancement:

| Gate | Checks |
|------|--------|
| Concept → Spec | No contradictions, dependencies listed, scope bounded |
| Spec → Plan | All fields typed, error cases specified, constraints explicit |
| Plan → Code | All spec sections covered, technology decisions documented |
| Code → Test | All contracts tested, error cases covered |
| Test → Review | All tests pass, no regressions |
| Review → Verify | Clean-context review passes, no blocking issues |
| Verify → Commit | All verification levels pass, user approval |

### Two-Stage Testing

- **Test** (Phase 5) — functional tests only (unit + mock), covering changed code
- **Verify** (Phase 7) — regression, integration, and live testing after review passes

If Verify finds issues: fix → Test → Review → Verify (cycle repeats).

### Pre-Commit Review

Code review is performed by a **clean-context subagent** — a fresh AI instance that hasn't seen the implementation process. This eliminates implementer blind spots and catches issues that the original author would miss:

- Spec compliance (all contracts, error cases, invariants implemented)
- Plan completeness (all tasks addressed)
- Project rules compliance
- SOLID principles (unless project rules override)
- Security, naming, code quality

### Language Independence

Concepts and specifications are **language-agnostic** — no programming languages, frameworks, or libraries mentioned. Implementation technology is chosen only in the Plan phase. This keeps design decisions clean and portable.

### Session Continuity

Work state is persisted in a **collaborative per-task** model so multiple AI
agents can work on the same task in parallel without conflicts:

- `.dev_flow/tasks/task_<ID>.md` — one file per task (source of truth). Shared
  between contributors. Holds Current Work Item, Description (shared), per-
  contributor Subtask blocks, Coordination Notes, Blocking Issues, Relevant
  Context, and a Shared Activity Log.
- `.dev_flow/active_context.md` — lightweight dashboard listing active tasks
  and recently completed ones (links to the task files).
- `.dev_flow/tasks/_index.md` — directory catalog with conventions and lists.

Resume anytime with `/dev-flow status` (lists active tasks) or
`/dev-flow status <task_id>` (details for one task).

**Collaboration rules:** each contributor owns its own **Subtask block** inside
a task file and its own tagged entries in shared sections. Contributors may
add new entries but never rewrite another contributor's content. Shared files
(dashboard, catalog, task headers) use **targeted edits** (single row/field)
and can be rebuilt from the task files when they drift. There is no
time-based ownership takeover — if a subtask stalls, add a new subtask block
referencing the original instead of editing it.

## File Structure

dev-flow creates the following structure in your project:

```
your-project/
├── docs/
│   ├── feature_name.concept.md    # Phase 1 output
│   ├── feature_name.sp.md         # Phase 2 output
│   ├── feature_name.plan.md       # Phase 3 output
│   └── feature_group.epic.md      # Coordinates 3+ related concepts
│
└── .dev_flow/
    ├── active_context.md           # Dashboard of active tasks
    ├── tasks/                      # Per-task context files (source of truth)
    │   ├── _index.md
    │   ├── task_C_AUTH.md
    │   ├── task_20260520_143022_refactor-login.md
    │   └── ...
    ├── session_history/            # Archived completed tasks
    ├── rules/                      # Project coding rules
    │   ├── _index.yaml
    │   ├── naming.md
    │   ├── structure.md
    │   └── ...
    └── skills/                     # Project technology knowledge
        ├── _index.yaml
        └── {domain}/
            └── {skill}.md
```

## Skill Structure

```
dev-flow/
├── SKILL.md              # Main skill definition and pipeline
├── phases/               # 15 phase definitions
│   ├── concept.md
│   ├── specification.md
│   ├── plan.md
│   ├── implement.md
│   ├── testing.md
│   ├── review.md
│   ├── verify.md
│   ├── propagate.md
│   ├── onboard.md
│   ├── fix.md
│   ├── ask.md
│   ├── do.md
│   ├── rule.md
│   ├── skill.md
│   └── status.md
├── roles/                # 14 AI-DSL subagent roles
│   ├── concept-author.ai.md
│   ├── spec-author.ai.md
│   ├── plan-author.ai.md
│   ├── implementer.ai.md
│   ├── reviewer.ai.md
│   ├── tester.ai.md
│   ├── propagator.ai.md
│   ├── advisor.ai.md
│   ├── context-tracker.ai.md
│   ├── dev-flow-orchestrator.ai.md
│   ├── onboard-coordinator.ai.md
│   ├── onboard-analyzer.ai.md
│   ├── onboard-docgen.ai.md
│   └── onboard-rules-extractor.ai.md
├── templates/            # Document templates
│   ├── concept.md
│   ├── specification.md
│   ├── plan.md
│   ├── epic.md
│   ├── spike.md
│   ├── active_context.md       # Dashboard
│   ├── task_context.md         # Per-task file
│   └── tasks_index.md          # tasks/_index.md
├── references/           # Architecture guidelines
│   └── solid-architecture.md
└── examples/             # End-to-end walkthrough
    └── rate-limiter.md
```

## Key Principles

1. **Code is a derived artifact** — it flows from concepts and specs, not the other way around
2. **No undocumented changes** — every modification traces back to a design decision
3. **Gates prevent drift** — incomplete specs can't become incomplete code
4. **Fresh eyes before commit** — clean-context review catches what you missed
5. **Living docs, not dead docs** — propagation keeps documentation current
6. **Rules and skills accumulate** — project knowledge is captured and reused across sessions

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or IDE extension

## License

MIT

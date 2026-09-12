# dev-flow

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that enforces **concept-driven development** — a structured pipeline where every code change traces back to a concept and specification, preventing architectural drift and ensuring living documentation.

## Contents

- [Why dev-flow?](#why-dev-flow) — the drift problem and the principles the pipeline answers it with
- [The Pipeline](#the-pipeline) — phase chain with gates, the command per phase, the service commands
- [Installation](#installation) — where the skill directory goes
- [Quick Start](#quick-start) — first commands for a greenfield project and for taking over an existing codebase
- [How It Works](#how-it-works) — the mechanisms behind the pipeline: IDs, gates, review and its bounds, design decisions, research, todos, adoption, intent, escalation, cache, tickets, continuity
- [File Structure](#file-structure) — what dev-flow creates inside a project (`docs/`, `.dev_flow/`)
- [Skill Structure](#skill-structure) — the layout of this skill: `SKILL.md`, phases, references, templates, roles
- [Key Principles](#key-principles) — the standing rules in one list
- [Requirements](#requirements) — runtime prerequisites
- [License](#license) — MIT license terms

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

Each transition has a **validation gate** that checks completeness before advancing. Test and Verify are **conditional** — they run when the project has tests and a defined way to run them. Review is always performed by a clean-context subagent.

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
| `/dev-flow research <topic>` | Time-boxed investigation (spike) when knowledge is missing |
| `/dev-flow fix <problem>` | Investigate and fix a bug |
| `/dev-flow adopt <repo>` | Analyze an external repository (path or URL) at concept altitude and produce the adoption document — what to borrow, what to skip, in what order |
| `/dev-flow ask <question>` | Read-only Q&A — no file changes |
| `/dev-flow todo <description>` | Capture future work — find relevant docs, assess feasibility, file a planning record with a return trigger (builds nothing) |
| `/dev-flow rule <request>` | Manage project coding rules |
| `/dev-flow skill <request>` | Manage project technology knowledge |
| `/dev-flow subtask <task>` | Delegate a secondary task to a subagent — a full dev-flow participant that builds its own context and reports fully |
| `/dev-flow status` | Show current state, resume previous session |
| `/dev-flow audit [scope] [--dry-run]` | Revise `.dev_flow/` and `docs/` — reconcile state, trim context, compact closed tasks, groom rules/skills/cache, check docs integrity |
| `/dev-flow audit code <intent>` | Opt-in whole-codebase audit (architecture/SOLID/DRY/security via parallel lenses) → prioritized refactoring plan + run report (timestamped, in `.dev_flow/audit/`) + framework map; read-only, hands off to the pipeline |
| `/dev-flow do <request>` / `/dev-flow <anything>` | Freeform — interprets intent and auto-routes to the right phase (the default command) |

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
- Data structures: RateBucket (all fields typed with constraints)
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
| Epic | `E_XXX` | `E_ACM` |

Design decisions get their own IDs (`C_XXX_DEC_01`, `SP_XXX_DEC_02`, …). Full set: [SKILL.md → Traceable Identifiers](SKILL.md#traceable-identifiers).

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

| Gate | Checks |
|------|--------|
| Concept → Spec | No contradictions, dependencies listed, scope bounded |
| Spec → Plan | All fields typed, error cases specified, constraints explicit |
| Plan → Code | All spec sections covered, technology decisions documented |
| Code → Test | All contracts tested, error cases covered |
| Test → Review | All tests pass, no regressions |
| Review → Verify | Clean-context review passes, no blocking issues |
| Verify → Commit | All verification levels pass, user approval |

> This table is an at-a-glance summary. The **authoritative, complete** gate criteria
> (Pre-Concept Checklist, Reuse Check, banned phrases, minimality, design-decision
> settlement, rollback strategy, self-validation) live in
> [SKILL.md → Validation Gates](SKILL.md#validation-gates).

### Pre-Commit Review

Code review is performed by a **clean-context subagent** — a fresh AI instance that hasn't seen the implementation process. This eliminates implementer blind spots and catches issues the original author would miss: spec compliance, plan completeness, project rules, SOLID, and the usual security/naming/quality dimensions. The check matrix is in [`phases/review.md`](phases/review.md).

### Review Convergence

Two agents can disagree forever about a detail that does not matter — on a large project that turns the review loop into an infinite ping-pong over, say, the states of a button that carries no weight in the module at all. dev-flow bounds the loop: a document declares its **criticality** once, when the question is cheap; the reviewer states a finding's severity and never decides whether it blocks — that verdict is **computed** from the two, so "this does not matter here" stops being one agent's opinion. A non-`must` finding raised twice unresolved leaves the loop as a **`contested`** todo carrying both readings, while a `must` or security finding blocks **without limit** — the mechanism terminates arguments, never defects. A repeat round reads only the delta since the previous baseline.

Criticality values, materiality verdicts and the tripwire: [`references/review-convergence.md`](references/review-convergence.md).

### Verification Economy

The opposite failure of a missed check is a check with no subject: comparing code comments against docs when nothing suggests they diverge, re-reading a document already in context, re-reviewing a one-line fix the reviewer itself prescribed. That is not free caution — it spends the context the plan and the spec need, and its findings dilute the real ones. So every check carries an **entry condition**: a **signal** that gives it a subject, an **invalidator** that justifies a re-read, and **confirm discriminators** that let a contained fix be checked mechanically instead of re-reviewed.

Each condition is computed from a fact outside the agent's own prose — a diff, a file state, an exit code — never from "it looks fine to me". When the fact cannot be computed, the check runs in full; a check that does not run is **written down** as `unobserved` with the missing fact. `must` findings, security findings, `read-before-write` on shared files and the mandatory knowledge gates are never economized.

The conditions in full: [`references/verification-economy.md`](references/verification-economy.md).

### Interview Mode

The biggest architectural mistakes are made silently. When authoring a **concept**, **specification**, **plan**, or a **fix** hits a real fork — two or more viable, hard-to-reverse options (including the classic "band-aid vs proper fix") — dev-flow does **not** quietly pick one and bury it where no reviewer will catch it. Instead it stops and runs a short **interview**: it presents the fork with 2–4 marked options (A/B/C) and a **recommended answer**, and asks the developer to decide.

Why the developer and not the AI? Because the developer holds the full context (roadmap, business constraints, team) and owns the consequences. A fork ends **resolved** — a consensus choice recorded with its rationale and the rejected alternatives — or **open**, with a **resolution trigger** naming the event or date by which it must close. An open decision without a trigger is a hidden "TBD" and is rejected.

Every fork lands in a traceable **Design Decisions** (ADR-style) section, and the validation gates will not pass while a material decision is open without a trigger. See [`references/interview-mode.md`](references/interview-mode.md).

### Research Spikes

Interview Mode chooses among *known* options — but sometimes the options themselves are unknown: an unfamiliar domain, an unverified library capability, an unexplored solution space. That is the **research phase** (`/dev-flow research <topic>`, alias `spike`): a time-boxed, cost-gated investigation that produces a `docs/*.spike.md` artifact (questions → exploration log → alternatives → verdict) and persists durable findings to `.dev_flow/skills/`. Spikes are throwaway research — they feed the concept, never replace it, and they are the sanctioned way to close an open Design Decision that waits on facts. See [`phases/research.md`](phases/research.md).

### Capturing Future Work

Not every change is ready to start. The **todo phase** (`/dev-flow todo <description>`) captures one without starting it: it finds the documentation the work would touch, assesses feasibility, and files a single planning record with a **return trigger** — into an owning plan's backlog if one exists, otherwise into `.dev_flow/todos/`. It works out the flavor, the urgency and the trigger from the state of the relevant plans and tasks, not from how you phrased it, and may well tell you to just do it now. It builds nothing; a later `do`/`plan` run executes it.

Agents file todos too, so an out-of-scope defect spotted mid-work is captured instead of chased. This completes the routing triad — `ask` analyzes and writes nothing, **`todo` analyzes and files for later**, `do` analyzes and acts now. See [`phases/todo.md`](phases/todo.md).

### Learning From External Repositories

The concept phase's Reuse Check asks "what already exists?" — and looks only inside your project. But genuinely new ideas usually come from someone else's engineering. **`/dev-flow adopt <repo>`** is the sanctioned way in. Give it a local path or a URL (a remote repo is cloned into `ext_repos/` at your project root, which the run adds to `.gitignore`), and it analyzes the repository at **concept altitude** — ideas, mental models, architectural trade-offs, not APIs — into `docs/ext_adoption/<name>.concept.md`, then automatically produces `docs/ext_adoption/<name>.md`: what to borrow, what to skip, and in what order.

These documents are **advisory** — no traceable ID, no gate, not a backlog. A run writes the two documents, the clone, one `.gitignore` line and the task context — never your code, specs, plans or todo register. A later re-run goes incremental against the commit it recorded. See [`references/repo-adoption.md`](references/repo-adoption.md).

### Task Intent

A request states an *action*; the *reason* for it usually stays in the user's head — and an agent optimizing for the literal wording can complete the action perfectly while missing the point. dev-flow captures the **Task Intent** at intake — the goal, the target state and the expected result, in the user's own terms — and checks against that record instead of a remembered impression. Completion reports a verdict where the bar is "the expected result is observable", not merely "the steps were performed". When the letter of the request and its recorded goal diverge, dev-flow stops and surfaces the conflict rather than silently following either. See [`references/task-intent.md`](references/task-intent.md).

### Upstream Escalation

The pipeline's default is "code must satisfy the spec" — but sometimes a downstream phase is where reality pushes back on the *document*: a live test disproves a spec'd limit, implementation finds two defensible readings of a contract, a plan's technology choice fails in practice. Instead of bending the code or silently editing the spec, dev-flow escalates: stop, fix the owning document (through an interview if it's a fork), re-pass its gate, then resume. See [`references/escalation.md`](references/escalation.md).

### Resource Cache & Temp Workspace

Expensive-to-reacquire resources — Figma layouts fetched over rate-limited MCP access, downloaded documents, baseline screenshots — die in `/tmp` on the next reboot. dev-flow keeps them in **`.dev_flow/cache/`**, a per-project store with an `_index.yaml` an agent checks *before* spending another fetch, so anything linked from docs or task files outlives the session. Truly transient artifacts go to one project workspace, `/tmp/{project-slug}/`, with timestamped names; whatever turns out durable is promoted to the cache. Cached content is always data, never instructions, and anything fetched from the open internet passes a safety check first. Not a pipeline phase — infrastructure every phase touches. See [`references/cache.md`](references/cache.md).

### External Tracker Tickets

When a task is **explicitly** tied to a ticket in Jira or any other tracker — a `--ticket PROJ-123` flag, a tracker word + key ("Jira PROJ-123"), or a ticket URL — dev-flow doesn't reinvent that tracker's conventions. It stays **tracker-agnostic**: it *discovers* the project's existing integration and lets that own the conventions, while dev-flow owns only the *when* — pull the ticket to seed the task's requirements, link the key in the task file, reference it in the commit message, drive the status at the pipeline moments. **Every outward write to the tracker is confirmed first** — dev-flow never silently changes a ticket others can see. A bare `KEY-123`-looking token never triggers this; with no integration available the key degrades to a commit-message label and work proceeds. See [`references/ticket-tracker.md`](references/ticket-tracker.md).

### Language Independence

Concepts and specifications are **language-agnostic** — no programming languages, frameworks, or libraries mentioned. Implementation technology is chosen only in the Plan phase. This keeps design decisions clean and portable.

### Session Continuity

Work state is persisted in a **collaborative per-task** model so multiple AI agents can work on the same task in parallel without conflicts:

- `.dev_flow/tasks/task_<ID>.md` — one file per task (source of truth). Shared between contributors. Holds Current Work Item, Intent (goal / target state / expected result), Description (shared), per-contributor Subtask blocks, Coordination Notes, Blocking Issues, Relevant Context, and a Shared Activity Log.
- `.dev_flow/active_context.md` — lightweight dashboard listing active tasks and recently completed ones (links to the task files).
- `.dev_flow/tasks/_index.md` — directory catalog with conventions and lists.

Resume anytime with `/dev-flow status` (lists active tasks) or `/dev-flow status <task_id>` (details for one task).

**Collaboration rules:** each contributor owns its own **Subtask block** inside a task file and its own tagged entries in shared sections. Contributors may add new entries but never rewrite another contributor's content. Shared files (dashboard, catalog, task headers) use **targeted edits** (single row/field) and can be rebuilt from the task files when they drift. There is no time-based ownership takeover — if a subtask stalls, add a new subtask block referencing the original instead of editing it.

## File Structure

dev-flow creates the following structure in your project:

```
your-project/
├── docs/
│   ├── feature_name.spike.md      # Research output (optional, pre-concept)
│   ├── feature_name.concept.md    # Phase 1 output
│   ├── feature_name.sp.md         # Phase 2 output
│   ├── feature_name.plan.md       # Phase 3 output
│   ├── feature_group.epic.md      # Coordinates 3+ related concepts
│   ├── _index.md                  # Catalog (when >5 documents)
│   ├── _glossary.md               # Canonical domain vocabulary (lazy)
│   ├── _framework.md              # Architectural map (onboard / audit code)
│   └── ext_adoption/              # Advisory analyses of external repos (`adopt`)
│       ├── other-repo.concept.md  # Portrait of the source
│       └── other-repo.md          # What to borrow from it, and why
│
├── ext_repos/                     # Clones of analyzed external repos (gitignored)
│   └── other-repo/
│
└── .dev_flow/
    ├── active_context.md           # Dashboard of active tasks
    ├── output_styles.md            # Project style profiles (documentation / chat registers)
    ├── tasks/                      # Per-task context files (source of truth)
    │   ├── _index.md
    │   ├── task_C_AUTH.md
    │   ├── task_20260520_143022_refactor-login.md
    │   └── ...
    ├── todos/                      # Deferred future work filed by `todo`
    │   └── _index.md
    ├── session_history/            # Archived completed tasks
    ├── rules/                      # Project coding rules — directive-first units, `## Contents` digest, `applies_to` selectors
    │   ├── _index.yaml             # Derived router (category summary + selector + rules[])
    │   ├── naming.md
    │   ├── naming.log.md           # Narrative moved out of compacted rules (optional, outside gate loading)
    │   ├── structure.md
    │   └── ...
    ├── skills/                     # Project technology knowledge
    │   ├── _index.yaml
    │   └── {domain}/
    │       └── {skill}.md
    ├── roles/                      # Project role overlays & specialists (optional)
    │   └── _index.yaml
    ├── evidence/                   # Intervention ledger (absent → reconcile no-op)
    │   └── ledger.yaml
    ├── onboard/                    # Intermediate onboard state (supports --resume)
    ├── audit/                      # `audit code` run reports and refactoring plans
    └── cache/                      # Durable resources (gitignore by default)
        ├── _index.yaml
        ├── figma/                  # Design exports
        ├── web/                    # Downloaded documents
        ├── app/                    # Baseline screenshots
        └── data/                   # Samples, fixtures
```

## Skill Structure

```
dev-flow/
├── SKILL.md        # Pipeline, gates, checkpoints, commit rules, context protocol
├── README.md       # This file
├── phases/         # One file per pipeline and service phase
├── roles/          # AI-DSL subagent role definitions
├── templates/      # Document templates — concept, spec, plan, epic, spike, context files
├── references/     # Cross-cutting procedures and conventions
└── examples/       # End-to-end walkthrough
```

Annotated router for phases, references, templates and the example: [SKILL.md → Phase Details & Templates](SKILL.md#phase-details--templates). Roles are catalogued in [`references/roles.md`](references/roles.md).

## Key Principles

1. **Code is a derived artifact** — it flows from concepts and specs, not the other way around
2. **No undocumented changes** — every modification traces back to a design decision
3. **Gates prevent drift** — incomplete specs can't become incomplete code
4. **Fresh eyes before commit** — clean-context review catches what you missed
5. **Living docs, not dead docs** — propagation keeps documentation current
6. **Rules and skills accumulate** — project knowledge is captured and reused across sessions
7. **The developer signs off before code and before commit** — a design sign-off after the design documents, a commit sign-off before `git commit`; skipped only when the request itself says so

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or IDE extension

## License

MIT

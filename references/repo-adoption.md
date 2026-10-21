# External Repository Adoption — the `adopt` Command

The service command that brings external prior art into the pipeline with a named source, a pinned commit, and a relevance verdict. It analyzes an external repository at **concept altitude** and automatically produces an adoption document for the current project.

Adoption documents are **advisory**: no traceable ID, no validation gate, not a backlog. They are input to whoever authors the concept — never authorization for a change. A concept grown from one cites it as its origin; nothing depends on it in the other direction.

`adopt` is a service command, not a pipeline stage. It receives nothing from a gate and hands nothing to one.

## Contents

- [Command](#command) — `/dev-flow adopt <repo>` with `--subpath` and `--readopt`, recognized aliases, freeform routing via do
- [What a run may write](#what-a-run-may-write) — Closed write list (adoption concept + document, `ext_repos/`, `.gitignore`, `.dev_flow/**`), what is never written, and the working-directory rule
- [Procedure](#procedure) — Steps 1–9: resolve, acquire workspace with the source stamp, run mode, source analysis, target context with records and the reconciliation of the previous document, verdicts, synthesis, write, report with the source check
- [Edge cases](#edge-cases) — Situation → behaviour table: self-analysis, unreachable URL, non-git source, dirty clone, collisions, monorepo, large source, previous document, rejected item, problem with no record
- [Delegation](#delegation) — Steps 4 and 6 may go to a read-only clean-context subagent, launched from the target root, returning concepts or verdicts, never files
- [Boundary with the resource cache](#boundary-with-the-resource-cache) — Clones live in `ext_repos/`, not `.dev_flow/cache/`; `audit cache` does not curate them

## Command

```
/dev-flow adopt <repo>                      # local path, URL, or the name of a tracked repo
/dev-flow adopt <repo> --subpath <dir>      # narrow the analysis to a monorepo subdirectory
/dev-flow adopt <repo> --readopt            # re-derive the adoption document even if the source has not moved
```

`analyze-repo` and `adoption` are recognized aliases. Freeform intent ("розбери репозиторій X", "what's worth taking from Y") routes here through [do](../phases/do.md).

## What a run may write

A closed list of writes in the target project. Anything outside it is a defect, not a judgement call.

| Path | Operation | Condition |
|------|-----------|-----------|
| `docs/ext_adoption/<name>.concept.md` | create \| update | always |
| `docs/ext_adoption/<name>.md` | create \| update | always |
| `ext_repos/<name>/` | create | only when the run cloned it |
| `.gitignore` | append one line | only when `ext_repos/` is not already ignored and the target is a git repo |
| `.dev_flow/**` | update | the ordinary task-context bookkeeping every phase does |

No code, no specs, no plans, no `docs/_index.md`, no `.dev_flow/todos/` entry. Filing the high-relevance items as todos is the developer's call — the run names them in its report and stops.

The source repository is never modified. `git pull` on a clone is the only write toward it, under the conditions in Step 2. Never change the working directory into the source worktree: address it by absolute path or with `git -C <worktree>`. Step 9 checks the source by command.

## Procedure

### Step 1 — Resolve the reference

| Input form | Resolution |
|-----------|-----------|
| URL | `ext_repos/<name>`, cloned in Step 2. `<name>` = last URL segment minus `.git`, kebab-cased |
| Local path | Used in place. Not cloned, not pulled |
| Name of a tracked repo | `ext_repos/<name>`, refreshed in Step 2 |

If the name is already taken in `ext_repos/` by a **different** origin, use `<owner>-<repo>` and say so — never overwrite an existing clone. The same origin is not a collision: it is the tracked repo.

**Stop before any write if** the source worktree is the target project root, the target lives inside the source, or the source lives inside the target anywhere other than `ext_repos/`. Report both paths. Nesting is allowed only under `ext_repos/` — that is what a clone is; a subdirectory of the project itself is self-analysis even when it carries its own `.git` (a submodule, a docs subrepo).

### Step 2 — Acquire the workspace

Ensure `ext_repos/` is ignored *before* cloning: if `git check-ignore ext_repos/` fails, append the line (creating `.gitignore` if absent). Test the pattern's effect, not the literal line — a broader pattern that already covers it is enough. A target that is not a git repo skips this step and the report says so.

Then:

- **Cloned** — clone into `ext_repos/<name>`. A clone failure (unreachable, private, network) stops the run with git's own message; never infer a repository's contents from its name.
- **Tracked** — `git -C <worktree> pull` only when the worktree is clean. A dirty tree skips the pull, analyzes the current state, and reports the fact. Never stash, reset, or checkout in someone else's repository.
- **Local path** — no refresh at all. The user's own worktree is not the command's to move.

Record `<name>`, the HEAD commit, and the origin URL. A non-git directory records no commit — analysis still works, incrementality does not.

Then, whatever the refresh did — also when it did nothing — create the source stamp once per run, outside the worktree, and keep its path in working memory: `mkdir -p /tmp/{project-slug}/scratch && touch /tmp/{project-slug}/scratch/adopt-<name>_<YYYYMMDD_HHMMSS>.stamp`. Step 9 checks the source against it.

### Step 3 — Pick the run mode

| Mode | Condition | Effect |
|------|-----------|--------|
| `CREATE` | no source analysis exists, **or** either side has no commit (a non-git source) | Full analysis |
| `UPDATE` | analysis exists, both commits present, and they differ | Incremental over the commit range |
| `METADATA` | analysis exists, both commits present and equal | Refresh the analysis date; leave the body untouched |

`METADATA` skips re-deriving the adoption document too — unless `--readopt` was given.

### Step 4 — Produce the source analysis

**`CREATE`** — read in priority order, each group only if present:

1. `README*`, `CLAUDE.md`, `AGENTS.md`, agent rule files
2. Documentation: `docs/`, `ARCHITECTURE`, `DESIGN`, `SPEC`, `PROTOCOL`, ADRs
3. Structure: root manifests, top-level directories, key abstractions, entry points
4. Supporting: `CHANGELOG` (direction of travel), examples, test layout

Then extract the concepts — the abstractions driving the project, its mental models and metaphors, how control and data are organized, its design philosophy; the architectural ideas — layers, interaction patterns, extensibility, state, trade-offs; and what makes it unusual — non-standard choices, innovations, counterintuitive decisions.

Stay at concept altitude. Describe **ideas**, not APIs, classes, functions, or line numbers. Every concept states *why* it exists, not only what it is. Descend into a specific file only as a point check on an idea already named — the bottom-up module walk belongs to [Code Audit](code-audit.md) and is not repeated here.

An empty or unreadable source stops the run. Do not manufacture concepts from a directory listing.

**`UPDATE`** — diff `--name-status` over the commit range, then weight the changes:

| Weight | What | Action |
|--------|------|--------|
| High | `README`/`ARCHITECTURE`/`DESIGN`/`SPEC` changes, new top-level directories, project manifest changes, documentation changes, key abstraction changes | Read |
| Medium | New modules or packages, significant restructuring, new examples or templates | Read |
| Low | Bug fixes in individual files, test changes with no interface change, CI/CD, dependency bumps | Skip |

Resolve each existing concept to `NEW` / `MODIFIED` / `REMOVED` / `NONE`, apply a **minimal diff** — untouched concepts stay byte-identical — and append a row to the analysis's update history.

### Step 5 — Collect the target context (read-only)

Whatever exists of: `docs/_index.md`, `docs/*.concept.md` / `*.sp.md` / `*.plan.md`, `docs/_glossary.md`, `docs/_framework.md`, `.dev_flow/rules/_index.yaml`, `.dev_flow/skills/_index.yaml`, `.dev_flow/tools/_index.yaml`, root `README`/`CLAUDE.md`/`AGENTS.md`, the dependency manifest, the top-level layout, and the entry points.

Extract the traceable ID of everything the adoption document will cite. A missing source is not an error — collect what is there.

Find the records of problems by search, not by reading whole files: `.dev_flow/tasks/`, `.dev_flow/session_history/`, `.dev_flow/todos/`, `.dev_flow/rules/`, `.dev_flow/evidence/`, `docs/`. A record is only one of these:

- an incident in a task file or in `session_history/`;
- a Review Rounds row;
- an intervention-ledger entry;
- the `evidence.ref` of a rule;
- a DEC record;
- a todo;
- a document banner or Changelog row;
- a step of this run that met the condition.

Not a record: missing protection that a search confirms, a plausible problem, a claim of the source.

**Reconcile the previous adoption document** of the same source, when it exists. Each of its `high` and `medium` items and each derived idea gets exactly one state. The first match wins:

| State | Condition | Evidence |
|-------|-----------|----------|
| `shipped` | The target implements the item fully | A commit, a shipped file, or a DEC record |
| `rejected` | The target took the item or a part of it and then refused it | A DEC record, a document banner, a review row, or a `session_history/` or task-file entry about the refusal; name the shipped part, if any |
| `partially` | The target implements part of the item and refused none of it | A commit, a shipped file, or a DEC record, plus what is missing |
| `open` | No trace of implementation or refusal | Optional |

An item whose concept the source removed still gets a state.

### Step 6 — Assess relevance

Every concept in the source analysis gets exactly one verdict.

First: a concept that matches a `rejected` item of the reconciliation is `low`, and its reason points to the refusal record. It goes higher only with a record of the problem newer than the refusal, named in the reason.

| Level | Test | Record |
|-------|------|--------|
| `high` | Solves a known problem of the target — one with a record in the target (Step 5) — fits the stack, has a concrete integration point, and the effort-to-value ratio is favourable | Where it lives in the source (1–3 paths) · which target problem or traceable ID it attaches to · the record of the problem · what exactly to take · effect, as the upper bound the record supports, and cost · which target contracts it risks |
| `medium` | Useful but needs an architectural choice, conflicts with an existing concept, depends on something unbuilt, or the problem has no record in the target | The same, with the record when one exists, plus an explicit ruling: **take / defer / decline**. With no record of the problem: only **defer** with the trigger "first observed case", or **decline** |
| `low` | Outside the target's scope, contradicts its non-goals, or already solved another way | Name and one sentence of reason. Nothing more |

A `medium` without a ruling is invalid: "needs a decision" with no decision is deferred work with no owner. A **defer** names its trigger.

Sort `high` + `medium` by effect-over-cost, mark blockers and inter-item dependencies, and put the cheapest win first.

### Step 7 — Synthesize derived ideas

Ideas that arise at the intersection of the source's concepts and the target's current gaps, but that the source does **not** already solve. Each carries an `Inspired by` naming what prompted it, and they live in their own section — synthesis must never read as borrowing.

An idea that matches a `rejected` item of the reconciliation is left out. It comes back only with a record of the problem newer than the refusal, named in the idea.

### Step 8 — Write the adoption document

Sections, in this order: purpose · what the target already has · most relevant · medium relevance · less relevant / don't take · derived ideas · recommended order of work · sources · changelog.

Header carries: the advisory-document declaration, a relative link to the source worktree, a relative link to the source analysis, the upstream URL, the analysis date with the source commit, the target project name and path, and the target documents the body cites.

Writing rules:

- **Language** follows the target project's documentation language.
- **All paths are relative markdown links** from the adoption document's own location.
- **Cite the target's traceable IDs** where they exist — the document must connect to what is already there, not float beside it.
- **Updating an existing document** preserves its changelog and appends a dated row. Concepts unchanged since the last run keep their text.
- **The "what the target already has" section carries the reconciliation** from Step 5, with its evidence.
- **An empty `high` section is a valid result** — write it, and say so plainly. Never promote a `medium` to make the document look useful.

### Step 9 — Report

Both artifact paths · run mode and whether the adoption document was created, updated, or left unchanged · `<name> @ commit` with origin and how the worktree was acquired · how the verdicts distributed · the cheapest high-relevance win (or a plain statement that there is none) · and the boundary: what was written, and that nothing outside it was — including that no todo was filed · and the source check, with its command and output:

```
find <worktree> -name .git -prune -o -newer <stamp> -print
```

- Empty output — nothing in the worktree, outside `.git`, changed after the stamp.
- Listed paths — each is a change to the source during the run. Name each in the report and remove nothing: the developer decides.
- No stamp, or the check did not run — the source claim is `unobserved`; give the reason.

Offer, without doing it, the next steps that make sense: adding the document to `docs/_index.md`, and the pipeline phase each high-relevance item would start from.

## Edge cases

| Situation | Behavior |
|-----------|----------|
| Target is the source, or the target sits inside the source | Stop before any write; name both paths |
| Source is inside the target but not under `ext_repos/` (`adopt ./src`, `adopt ./docs`) | Stop — that is self-analysis, even if the directory has its own `.git`. Clone or copy it into `ext_repos/` to analyze it |
| URL unreachable or private | Stop with git's message verbatim |
| Path exists but is not a git repo | Analyze; record no commit; the next run will be a full one |
| Clone has uncommitted changes | Skip the pull, analyze the current state, report it |
| Source unchanged since the last analysis | `METADATA`; the body stays untouched; `--readopt` re-derives the adoption document against the target's current state |
| URL given for a repo already tracked under the same origin | Not a collision — treat as tracked, refresh in place. Never clone a second copy under a suffixed name |
| Name collision under a different origin | Clone as `<owner>-<repo>`; leave the existing one alone |
| Monorepo subdirectory named | Clone the whole repo, analyze the subdirectory, pin the repository commit, record the narrowing |
| `.gitignore` absent | Create it with the one line |
| `ext_repos/` already ignored, by any pattern | Change nothing |
| Target is not a git repo | Skip the ignore step; note it in the report |
| Source too large to read whole | Narrow to the named subsystems and record the narrowing, with its reason, in the analysis header |
| Previous adoption document of the same source exists | Reconcile each of its `high` / `medium` items and each derived idea (Step 5); write the reconciliation into the "what the target already has" section |
| Concept matches an item the target rejected | `low` with a pointer to the refusal record; higher only with a newer record of the problem, named in the reason |
| Problem is plausible, but the target has no record of it | At most `medium` with **defer** and the trigger "first observed case", or **decline**. A search that confirms missing protection is not a record |
| Adoption requested with no analysis | Run `CREATE` first — relevance cannot be assessed against concepts nobody has extracted |

## Delegation

Steps 4 and 6 are the expensive ones. When the source is large enough to flood the main context, hand each to a **read-only clean-context subagent** briefed from this procedure — its return is the extracted concepts, or the verdicts, never the files it read. Launch it only while the working directory is the target root, and brief it to address the source by absolute path, never by `cd`. No base role ships for it: brief it with the relevant step above plus the target context from Step 5. See [Delegation for Focus](delegation.md).

## Boundary with the resource cache

Clones live in `ext_repos/`, **not** in the [resource cache](cache.md), and `audit cache` does not curate them: a clone is a live worktree that refreshes itself, not an immutable fetched artifact under a freshness policy.

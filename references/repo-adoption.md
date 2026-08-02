# External Repository Adoption — the `adopt` Command

The service command that brings external prior art into the pipeline with a named source, a pinned commit, and a relevance verdict. It analyzes an external repository at **concept altitude** and automatically produces an adoption document for the current project.

Adoption documents are **advisory**: no traceable ID, no validation gate, not a backlog. They are input to whoever authors the concept — never authorization for a change. A concept grown from one cites it as its origin; nothing depends on it in the other direction.

`adopt` is a service command, not a pipeline stage. It receives nothing from a gate and hands nothing to one.

## Command

```
/dev-flow adopt <repo>                      # local path, URL, or the name of a tracked repo
/dev-flow adopt <repo> --subpath <dir>      # narrow the analysis to a monorepo subdirectory
/dev-flow adopt <repo> --readopt            # re-derive the adoption document even if the source has not moved
```

`analyze-repo` and `adoption` are recognized aliases. Freeform intent ("розбери репозиторій X", "what's worth taking from Y") routes here through [do](../phases/do.md).

## What a run may write

A closed list. Anything outside it is a defect, not a judgement call.

| Path | Operation | Condition |
|------|-----------|-----------|
| `docs/ext_adoption/<name>.concept.md` | create \| update | always |
| `docs/ext_adoption/<name>.md` | create \| update | always |
| `ext_repos/<name>/` | create | only when the run cloned it |
| `.gitignore` | append one line | only when `ext_repos/` is not already ignored and the target is a git repo |
| `.dev_flow/**` | update | the ordinary task-context bookkeeping every phase does |

No code, no specs, no plans, no `docs/_index.md`, no `.dev_flow/todos/` entry. Filing the high-relevance items as todos is the developer's call — the run names them in its report and stops.

The source repository is never modified. `git pull` on a clone is the only write toward it, under the conditions in Step 2.

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
- **Tracked** — `git pull` only when the worktree is clean. A dirty tree skips the pull, analyzes the current state, and reports the fact. Never stash, reset, or checkout in someone else's repository.
- **Local path** — no refresh at all. The user's own worktree is not the command's to move.

Record `<name>`, the HEAD commit, and the origin URL. A non-git directory records no commit — analysis still works, incrementality does not.

### Step 3 — Pick the run mode

| Mode | Condition | Effect |
|------|-----------|--------|
| `CREATE` | no source analysis exists, **or** either side has no commit (a non-git source) | Full analysis |
| `UPDATE` | analysis exists, both commits present, and they differ | Incremental over the commit range |
| `METADATA` | analysis exists, both commits present and equal | Refresh the analysis date; leave the body untouched |

Incrementality needs a commit on both ends. A non-git source has none, so every run of it is a full one.

`METADATA` skips re-deriving the adoption document too — unless `--readopt` was given. The target project moves faster than the source, so re-deriving is available on request rather than charged on every run.

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

Whatever exists of: `docs/_index.md`, `docs/*.concept.md` / `*.sp.md` / `*.plan.md`, `docs/_glossary.md`, `docs/_framework.md`, `.dev_flow/rules/_index.yaml`, `.dev_flow/skills/_index.yaml`, root `README`/`CLAUDE.md`/`AGENTS.md`, the dependency manifest, the top-level layout, and the entry points.

Extract the traceable ID of everything the adoption document will cite. A missing source is not an error — collect what is there.

### Step 6 — Assess relevance

Every concept in the source analysis gets exactly one verdict.

| Level | Test | Record |
|-------|------|--------|
| `high` | Solves a known problem of the target, fits the stack, has a concrete integration point, and the effort-to-value ratio is favourable | Where it lives in the source (1–3 paths) · which target problem or traceable ID it attaches to · what exactly to take · effect and cost · which target contracts it risks |
| `medium` | Useful but needs an architectural choice, conflicts with an existing concept, or depends on something unbuilt | The same, plus an explicit ruling: **take / defer / decline** |
| `low` | Outside the target's scope, contradicts its non-goals, or already solved another way | Name and one sentence of reason. Nothing more — elaborating a rejection is wasted work |

A `medium` without a ruling is invalid: "needs a decision" with no decision is deferred work with no owner.

Sort `high` + `medium` by effect-over-cost, mark blockers and inter-item dependencies, and put the cheapest win first.

### Step 7 — Synthesize derived ideas

Ideas that arise at the intersection of the source's concepts and the target's current gaps, but that the source does **not** already solve. Each carries an `Inspired by` naming what prompted it, and they live in their own section — synthesis must never read as borrowing.

### Step 8 — Write the adoption document

Sections, in this order: purpose · what the target already has · most relevant · medium relevance · less relevant / don't take · derived ideas · recommended order of work · sources · changelog.

Header carries: the advisory-document declaration, a relative link to the source worktree, a relative link to the source analysis, the upstream URL, the analysis date with the source commit, the target project name and path, and the target documents the body cites.

Writing rules:

- **Language** follows the target project's documentation language.
- **All paths are relative markdown links** from the adoption document's own location.
- **Cite the target's traceable IDs** where they exist — the document must connect to what is already there, not float beside it.
- **Updating an existing document** preserves its changelog and appends a dated row. Concepts unchanged since the last run keep their text.
- **An empty `high` section is a valid result** — write it, and say so plainly. Never promote a `medium` to make the document look useful; the scale stops meaning anything the first time it happens.

### Step 9 — Report

Both artifact paths · run mode and whether the adoption document was created, updated, or left unchanged · `<name> @ commit` with origin and how the worktree was acquired · how the verdicts distributed · the cheapest high-relevance win (or a plain statement that there is none) · and the boundary: what was written, and that nothing outside it was — including that no todo was filed.

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
| Adoption requested with no analysis | Run `CREATE` first — relevance cannot be assessed against concepts nobody has extracted |

## Delegation

Steps 4 and 6 are the expensive ones. When the source is large enough to flood the main context, hand each to a **read-only clean-context subagent** briefed from this procedure — its return is the extracted concepts, or the verdicts, never the files it read. No base role ships for it: brief it with the relevant step above plus the target context from Step 5. See [Delegation for Focus](delegation.md).

## Boundary with the resource cache

Clones live in `ext_repos/`, **not** in the [resource cache](cache.md), and `audit cache` does not curate them: a clone is a live worktree that refreshes itself, not an immutable fetched artifact under a freshness policy.

# Recovery Points — File Operations That Cannot Be Undone

Cross-cutting discipline of **every phase**, subagents included. Not a pipeline stage, no command. Before an operation changes or removes the existing content of a file, the path needs a **recovery point**: a declared exception class, a file the task itself created, a repository that holds the content, or a verified backup. A project with no repository gets a recommendation to create one. When the developer declines it or it cannot be created, the agent makes a backup if one fits the task; otherwise the operation is refused, with an argument the developer can act on. The exceptions are a declared, closed list that a project extends by its own rules.

## Contents

- [Operations](#operations) — the closed list of operations that need a point, and what is not one
- [Recovery point](#recovery-point) — the four kinds, when the check runs, the owning repository and its facts
- [Routes](#routes) — what happens when a path has no point: backup, commit, repository, refusal
- [Repository recommendation](#repository-recommendation) — once per task, when it is made, when creation is impossible
- [Repository creation](#repository-creation) — `git init`, `.gitignore` first, secrets out, the baseline commit
- [Recovery commits](#recovery-commits) — the baseline commit and a commit of pre-existing paths: approval, review, message
- [Backups](#backups) — when a backup fits, the store, the manifest, restore and removal
- [Refusal](#refusal) — the argument the developer gets, and what lifts a refusal
- [Exceptions](#exceptions) — the declared classes and project additions
- [Autonomy and subagents](#autonomy-and-subagents) — what `Autonomy: full` and a subagent do
- [Relation to the hard floor](#relation-to-the-hard-floor) — what stays with `irreversible-destruction`
- [Boundaries](#boundaries) — what this discipline does not change

## Operations

An operation needs a recovery point for every path in its right column. The list is closed.

| Operation | Paths that need a point |
|-----------|-------------------------|
| `edit` — any write into an existing file: a targeted edit, an append, a whole-content replacement, a truncation | the file |
| `delete` — removal of a file or of a directory tree | every file it removes |
| `move` — a rename or a move | the source, and a target path it replaces |
| `in-place run` — a tool that rewrites existing files: a formatter, a codemod, `sed -i`, a generator over existing files, a package manager's rewrite of a manifest, a project tool with declared writes | every file it may rewrite, listed before the run (its dry run, file list or declared writes) |
| `vcs discard` — a vcs command that removes or overwrites content no commit holds: working-tree or index changes, untracked files, stash entries — for example `reset --hard`, `reset --merge`, `checkout -- <path>`, `checkout -f`, `switch --discard-changes`, `restore`, `stash drop`, `stash clear`, `clean`, `hg revert`, `hg update -C`, `svn revert`, `jj restore`, `jj abandon`. A vcs command whose effect on uncommitted content is not known counts as one | every file whose content it discards; a stash entry is held by a ref (see [Backups](#backups) → Store). A discard of changes the task did not write is also on the [hard floor](#relation-to-the-hard-floor) |

Not an operation: a read; a new file at a path where nothing exists; a write inside a place the task created (its prototype worktree, a scratch directory); the removal of a task's backup store and its `refs/dev-flow/<task_id>/` refs on the developer's word ([Backups](#backups) → Removal).

## Recovery point

| Kind | The path has it when |
|------|----------------------|
| `exempt` | it belongs to an [exception class](#exceptions) |
| `task-written` | the task created the file at a path where nothing was. A file the task found present — untracked or ignored included — is not task-written, though the task's diff lists it |
| `vcs` | at the check, its owning repository tracks it, it has no uncommitted changes, and no index flag hides its working content |
| `backup` | the task saved a [backup](#backups) of its current content and verified it |

**When.** Before the task's first operation on the path. A later operation of the task on the same path needs no new point: the first one holds the state before the task. When another hand changed the content since the task's last operation on it — the content read right before the operation differs from what that operation left — check again.

**Owning repository** — the repository whose working tree holds the path; a nested repository or a submodule owns its own files. Facts, gathered for all paths of an action burst in one run ([Round-Trip Economy](round-trip-economy.md)):

- git — `git -C <dir> rev-parse --show-toplevel` (a repository), `git ls-files --error-unmatch -- <path>` (tracked), `git status --porcelain --ignored -- <paths>` (uncommitted, untracked, ignored), `git ls-files -v -- <paths>` (a lowercase tag — assume-unchanged — or `S` — skip-worktree — is no `vcs` point: the repository may not hold the working content);
- another vcs — its own status command: `hg status`, `svn status`, `fossil changes`; Jujutsu records the working copy at every `jj` command, so `jj status` itself makes the `vcs` point.

A path lies **in no repository** only when `git rev-parse` reports that it is not in a git repository and no vcs metadata lies in the path's directory or a parent (`.hg/`, `.svn/`, `.jj/`, `.bzr/`, `_darcs/`, `CVS/`, `.sl/`, `.pijul/`, `.fslckout`, `_FOSSIL_`). Metadata of another vcs claims the path; when its status command is missing or fails, or the metadata belongs to a vcs not listed in the facts above, the fact is unestablished. Any other failure — dubious ownership, permissions, a missing directory — is an unestablished fact too: no `vcs` point, and no repository recommendation; the refusal names the failing command, the missing program, or the unlisted vcs and its metadata.

## Routes

A path with no point takes the first route whose condition holds:

| Condition | Route |
|-----------|-------|
| The path lies in a repository that does not hold its content — untracked, ignored, with uncommitted changes, or hidden by an index flag | A [backup](#backups) when one fits. Otherwise, for a tracked or untracked path, the developer commits it — or approves your [recovery commit](#recovery-commits) of those paths only; a path hidden by an index flag first needs the developer to clear the flag (`git update-index --no-assume-unchanged` / `--no-skip-worktree`). An ignored path, or a declined commit → [refusal](#refusal) |
| The path lies in no repository | The [repository recommendation](#repository-recommendation) first. Created → the baseline commit gives the path a `vcs` point, and a path it leaves out takes the row above. Declined or impossible → a backup when one fits, otherwise [refusal](#refusal) |
| Whether the path lies in a repository is not established | A backup when one fits, otherwise [refusal](#refusal) that names that cause ([Owning repository](#recovery-point)) |

## Repository recommendation

The main agent recommends a repository once per task, at the first of: the design sign-off ([SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints)), or the first operation that needs it. A design sign-off after it shows the recorded answer. The recommendation names the paths, what a repository gives (every later change can be undone and reviewed), and what [creation](#repository-creation) does. Record the answer in Activity: `recovery: repository created at <root>` / `declined — "<quote>"` / `impossible — <fact>`. A declined answer holds for the task; the next task recommends again.

Creation is **impossible** when no vcs program is on the machine and the developer does not install one, when the path lies outside the project root (a file in the home directory, a system file — no repository is created there), when the location is not writable for the agent, or when a project rule or the developer's standing instruction says no repository.

## Repository creation

On the developer's yes, in the project root:

1. `git init`.
2. Write `.gitignore` first (extend one that exists):
   - secret files — `.env` and `.env.*` other than examples, private keys, credential files;
   - the paths of the classes `temporary`, `build-output`, `cache`, `dependencies`, and the process lock and pid files of `lock`, present in the tree — never a dependency lock file, a generated file the project keeps, or `.dev_flow/` as a whole;
   - the places dev-flow keeps git-ignored: `.dev_flow/profile/`, `.dev_flow/proto/`, `.dev_flow/backups/`, `.dev_flow/cache/` ([Resource Cache → Git](cache.md#git)), `.dev_flow/access/journal.log`, `ext_repos/` — the closed list of those places; a reference that declares a new one adds it here in the same change;
   - files over `large_file_max` = 50 MB.
3. Stage everything except the files the task has created or changed so far — they stay in the task's diff. Scan the content of every staged file for a [secret](proportionate-caution.md#secrets) — a password, a token, an API key, a private key, a connection string that holds one; a file that holds one, and a binary credential store (a keystore, `*.p12`, `*.pfx`, `*.jks`, a database dump), is unstaged and added to `.gitignore`.
4. Commit `Baseline before <task_id>` ([Recovery commits](#recovery-commits)).
5. Activity: `recovery: repository created at <root>, left out: <classes and paths>`, the secret-holding files among them. A left-out path is in a repository that does not hold it — its route is the first row of [Routes](#routes).

## Recovery commits

The baseline commit and a commit of pre-existing paths that the developer approves (the first row of [Routes](#routes)) hold no change of the task. They take no pre-commit review; the developer's yes to the creation or to the commit is the approval; their messages are `Baseline before <task_id>` and `Recovery point before <task_id>: <paths>`. The commit sign-off shows them among the unpushed commits. They never carry a file the task created or changed. Before a commit of pre-existing paths, scan the content of each path for a secret as in creation step 3; a path that holds one, or is a binary credential store, is not committed — it goes to [refusal](#refusal).

## Backups

**A backup fits** when all hold:

- the paths are known before the operation — an operation whose path set cannot be listed first does not fit;
- each is a regular file — not a device, a socket or a FIFO, not a database file a running process holds open — or a stash entry of a repository that can hold a ref;
- the task's backups stay within `backup_max` = 100 MB, or the developer allowed a larger one for the task;
- the store is protected: `.dev_flow/backups/.gitignore` holding `*` is written when the store is created, in every project — a new file where nothing was, no point needed; in a repository, `git check-ignore -q .dev_flow/backups/<task_id>/manifest.md` confirms it; with no repository, an existing file protects the store when it holds `*`; an existing one that does not ignore the store leaves it unprotected;
- the developer has not declined backups for the task.

A backup is made without a question; the report of the step names it.

**Store** — `.dev_flow/backups/<task_id>/files/<project-relative path>`; a path outside the project root goes under `files/_external/<absolute path>`. A stash entry is held by a ref named by its commit, never by its index (an index is reused after a drop): `<oid>` = `git rev-parse stash@{<n>}`, then `git update-ref refs/dev-flow/<task_id>/stash-<oid> <oid>`, verified when `git rev-parse` of the ref equals `<oid>`; restore with `git stash store -m "$(git log -1 --format=%s <ref>)" <ref>`; the task's `refs/dev-flow/<task_id>/` refs go with the store's removal. One copy per path per point: the re-check of [When](#recovery-point) adds a copy `<path>.<n>` with its own manifest line.

**Make** — copy with mode and times kept (`cp -p`), compare the sha256 of the copy with the source, append the manifest line, then run the operation. A mismatch → no operation; copy once more, and a second mismatch means the backup does not fit.

**Manifest** — `.dev_flow/backups/<task_id>/manifest.md`, append-only, one line per copy, no file content:

```
- <path> · sha256 <hex> · <bytes> B · <operation> · YYYY-MM-DD HH:MM · [agent-id]
- <ref> · commit <oid> · <operation> · YYYY-MM-DD HH:MM · [agent-id]
```

The second form is a stash entry's line.

A backup of a file that holds a secret is a write to a git-ignored place outside history — allowed, and the value is never repeated in a message ([Seen and emitted secrets](proportionate-caution.md#seen-and-emitted-secrets)).

**Restore** — on the developer's request, or to undo the task's own failed operation: copy the backup back and compare its sha256 with the manifest. The current content first gets its own point, unless the task wrote all of it.

**Removal** — only on the developer's word; it removes the store and the task's `refs/dev-flow/<task_id>/` refs. The task completion report names the store with its file count and size, and the refs, and offers removal ([status → On task completion](../phases/status.md#on-task-completion)).

## Refusal

When no route holds, the operation is not run. That line of work stops; the rest of the task goes on. The main agent tells the developer, in the `chat` register ([Output Styles](output-styles.md)):

- **Operation** — what, on which paths;
- **No recovery point** — per path: no repository, untracked, ignored, uncommitted changes, an index flag, or the fact not established (the failing command, the missing program, or the unlisted vcs and its metadata); the repository answer (`declined — "<quote>"`, `impossible — <fact>`, `not asked — Autonomy: full`);
- **Would be lost** — per path: its size, its last change time, what the file is;
- **Routes checked** — the repository; the backup and the condition that failed (path set unknown, size over `backup_max`, not a regular file, store unprotected, backups declined); the commit and why it failed (declined, not asked — `Autonomy: full`, the index flag not cleared, the path holds a secret — named, never its value — or is a binary credential store, an ignored path); the exception classes, none matching;
- **Options** — create a repository (what it does); commit the paths — not offered for a path that holds a secret or is a binary credential store; allow a backup or a larger limit for the task; add an exception class by a project rule; run the operation yourself — the exact command for your own terminal.

Activity: `recovery: refused <operation> <paths> — <reason>`. The developer's acceptance of the loss does not lift a refusal; a new route does — a repository, a commit, a fitting backup, or a project exception class.

## Exceptions

A path of these classes needs no point: its content regenerates, nobody needs it back, or — for `dev-flow-state` — dev-flow's own protocols change it only in the ways the row names. A path whose class is not established is not exempt.

| Class | Paths |
|-------|-------|
| `temporary` | The system temporary directory, the session workspace `/tmp/{project-slug}/` and the agent's scratch directory; editor swap and backup files (`*.swp`, `*~`, `.#*`); files the writing tool names temporary (`*.tmp`, `*.temp`) |
| `build-output` | What the project's build writes and regenerates from sources: its output directories (`build/`, `dist/`, `out/`, `target/`, the `bin/` and `obj/` of a .NET build), object and bytecode files (`*.o`, `*.class`, `*.pyc`, `__pycache__/`), coverage and test reports. A directory counts only when the build configuration writes it |
| `cache` | Tool caches that regenerate: `.cache/`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`, `.gradle/`, `.next/cache/`, `node_modules/.cache/`. Not `.dev_flow/cache/` — its sources may not be fetchable again |
| `dependencies` | Installed dependency trees the package manager restores from the project's manifest (and its lock file, when the tool keeps one): `node_modules/`, `.venv/`, `venv/`, a `vendor/` its package manager's manifest lists (`go.mod` with `vendor/modules.txt`, `composer.json` with `composer.lock`) |
| `lock` | Process lock and pid files: `*.pid`, `.~lock.*#`, a lock file a running tool holds. A dependency lock file (`package-lock.json`, `yarn.lock`, `poetry.lock`, `Cargo.lock`, …) only while its own tool rewrites it in its run — a hand edit or a removal takes the routes: pinned versions do not regenerate |
| `generated-config` | IDE and editor workspace state the IDE rewrites on its own (`.idea/workspace.xml`); an `*.iml` only when the build import generates it; a file whose header marks it generated ("generated", "do not edit") when the change runs its generator |
| `dev-flow-state` | Targeted edits and appends to `.dev_flow/` files by dev-flow's own protocols (task files, dashboard, catalogs, todos, ledger, session history, profile, cache index); a rebuild of a derived view from its sources (dashboard and catalog from the task files; the `_index.yaml` routers of rules, skills and tools — never `cache/_index.yaml`); the moves into `session_history/` that audit and status name; the writes of this discipline to `.dev_flow/backups/`; and exactly these deletions: audit Step 7g's `prev/` copy of a bound tool, and a prototype place removed on the developer's `discard` or after its promotion. Any other removal — a skill, a rule, `.dev_flow/onboard/` — or whole rewrite of a `.dev_flow/` file takes the routes |

**Project additions.** A project rule ([rule phase](../phases/rule.md)) adds a class — path globs and why their content regenerates or is not needed back — or narrows a shipped one (a `build/` that holds hand-written files). The rule is part of the relevant set like any other ([Project Knowledge Is Binding](../SKILL.md#project-knowledge-is-binding)).

## Autonomy and subagents

- **`Autonomy: full`** — nothing is asked: the repository recommendation and the commit question are skipped, a backup is made when one fits, otherwise the operation is refused. The final report carries each refusal and the recommendation.
- **Subagent** — runs the check. It makes a fitting backup into the task's store for a path in a repository that does not hold the content, for a path whose repository fact is not established, or in no repository once the task file records the repository answer `declined` or `impossible`, or under `Autonomy: full`. The repository question, the commit question and a refusal go to the initiator with the operation, the paths and the facts.

## Relation to the hard floor

- A `vcs discard` of changes the task did not write stays on the hard floor as `irreversible-destruction`: the developer's word now ([Proportionate Caution → Hard floor](proportionate-caution.md#hard-floor)), and the discarded content needs a recovery point first. A discard of only the task's own changes — the path had no uncommitted changes at the task's first operation on it, and no other hand changed it since — is not a consequence.
- Any other file operation: with a recovery point it has no `irreversible-destruction` consequence; without one it is refused here, never asked.
- Data outside files (a database, a remote system) stays with the hard floor and the [Safe Testing Principle](../phases/verify.md#safe-testing-principle).

## Boundaries

- The secret discipline, the privileged runner and the access channels stay as they are ([Proportionate Caution](proportionate-caution.md), [Access Channels](access-channels.md)).
- Audit's non-destructive principle stays: Step 7g's `prev/` deletion is in the `dev-flow-state` class; Step 7h's deletions take their point like any other — a committed test has its `vcs` point ([audit](../phases/audit.md#guiding-principles)).
- No gate criterion changes. The developer checkpoints stay as they are; the design sign-off carries the repository recommendation or its recorded answer.

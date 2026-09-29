# Proportionate Caution — The Hard Floor, Secrets, and the Proportionality Block

Cross-cutting discipline of **every phase**. Not a pipeline stage, no command. Size caution to the consequence of an action, not to the action. The **hard floor** is the closed set of consequences that always need the developer's word now; no declaration, manifest, or autonomy mode relaxes it. A seen secret is an incident only in a place where secrets never live, and the agent never puts a secret value there. A relaxation of a listed row kind leaves a row in the **proportionality block**, which the developer checkpoints show.

## Contents

- [Hard floor](#hard-floor) — the consequences that need the developer's word now, what is not a consequence, the unestablished fact, what the agent never settles alone
- [Asking now](#asking-now) — `proceed` / `relay` / `ask-now` per action and caller
- [Secrets](#secrets) — what a secret is; the closed list of places where secrets never live
- [Seen and emitted secrets](#seen-and-emitted-secrets) — the reaction per event and place: continue, incident, refused, pasted
- [Proportionality block](#proportionality-block) — the task-file section: record forms, row kinds, append, created records, showing, archive
- [Autonomy](#autonomy) — what `Autonomy: full` skips and what still stops the work
- [Boundaries](#boundaries) — what this discipline does not change

## Hard floor

An action with any of these consequences needs the **developer's word now** — not a row in the block, not a delegated decision. Classify from facts: which system, which data, whether a recovery exists.

| Consequence | The action has it when |
|-------------|------------------------|
| `real-data` | It writes, changes, deletes, or exports (out of the system, also into the agent's own output) data of real people — in an external system or copied from one to the local machine. Reading aggregates that hold no data of people does not count |
| `production-write` | It writes to production |
| `money` | It triggers a payment, a subscription, or billing — also for an entity the task created |
| `outward-message` | It sends an email, an SMS, a push notification, or a chat message to a real person |
| `irreversible-destruction` | It discards content that cannot be recovered: a reset of uncommitted changes (`reset --hard`, `checkout --`, `restore`, `stash drop`), or a file removal or a replacement of the whole content, when the current task did not write that content and vcs history does not hold it. Or it deletes data outside a confirmed test target, and that data cannot be recovered |
| `access-relaxation` | Without the developer's confirmation, it removes or bypasses a command gate, widens the permissions of the agent runtime, or creates or changes a proxy or an access-manifest entry |

**Not a consequence:**

- An edit of files inside the change the task makes — also a repeat edit of the same file, and an edit of a file the task created, with or without vcs.
- A change of committed files under vcs.
- A deletion of build artifacts, caches, and temporary files that regenerate.
- A deletion of entities and data the task recorded as created ([created records](#proportionality-block)).
- An action on a local database seeded from the project's own fixtures or migrations.

The created exemption applies to `irreversible-destruction` only: a payment for a subscription the task created is still `money`. A file that is new in the task's diff counts as created without a record. A file under vcs is recoverable only when it has no changes since the last commit.

**The classifier returns a set.** It can be empty. When an action has several consequences, each applies.

**Unestablished fact — `FACT_UNKNOWN`.** When a fact that a row needs is unknown (is this production, who receives the message, where the data came from, does a recovery exist), the row applies. The question to the developer names the unknown fact.

The hard floor also holds what the agent never settles alone:

| Category | Meaning |
|----------|---------|
| `must-rule` | A violation of a project `must` rule. The agent does not leave, defer, or accept it: it fixes it, or hands it to the developer |
| `security-defect` | A security defect. The same as `must-rule` |
| `secret-exposure` | Output or write of a secret value into a place where secrets never live. The agent refuses ([seen and emitted secrets](#seen-and-emitted-secrets)). A write to another place (a git-ignored file outside history) is allowed without a repeat of the value in the chat |

The developer checkpoints (design sign-off, commit sign-off) are not part of the hard floor: [SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints) owns them.

## Asking now

Decide per action, from its consequences and the caller:

| Consequences | Caller | Outcome |
|--------------|--------|---------|
| none | any | `proceed` |
| one or more | subagent | `relay` — do not perform the action; hand it to the initiator |
| one or more | main agent | `ask-now` — under any `Autonomy` |

`ask-now` is an [Interview Mode](interview-mode.md) question now, whatever its entry criteria say. It names the action, its consequences, and the unestablished facts. The answer covers the named action only.

`must-rule` and `security-defect` do not go through this decision. Their usual path is the fix.

## Secrets

**A secret** is a credential for a system outside the local machine or for the account of a person: a password, a token, an API key, a private key, a connection string that holds one. Not a secret: a default password of a local service that the project itself deploys (for example, a database in a development `docker-compose`); a stub or a fixture with no access to a real system.

**Places where secrets never live** — closed list:

| Site | A location is this site when |
|------|------------------------------|
| `vcs-tracked-or-history` | The file is under vcs in its owning repository, or the path is in the history (`git log --all -- <path>` is not empty) |
| `not-ignored-file` | The owning repository does not ignore the file (`git check-ignore` run in the repository that owns the path — a nested repository, a submodule) |
| `persisted-log` | A log or an output kept in `.dev_flow/`, in CI, or in a report that is passed on |
| `task-file` | `.dev_flow/tasks/`, `.dev_flow/todos/`, `.dev_flow/session_history/`, `.dev_flow/active_context.md` |
| `cache` | `.dev_flow/cache/` |
| `profile` | `.dev_flow/profile/` |
| `test-passport` | The passport of a test (the metadata that describes the test) |
| `access-manifest` | An access manifest |
| `agent-message` | Any message of the agent to the developer or to another agent |
| `command-line-argument` | An argument or an environment variable in a command line that the agent composes |

A project without vcs: the `vcs-tracked-or-history` and `not-ignored-file` rows do not apply.

## Seen and emitted secrets

In every case: never repeat, copy, or store the value — except the write to another place in the table below.

| Event | Where | Reaction | Outcome |
|-------|-------|----------|---------|
| Seen | A file that its owning repository ignores and that is not in history, or a local output that is kept nowhere | Continue. No stop, no alarm, no rotation proposal, no review finding, no row in the block. This includes the usual `.env` with the developer's keys | `continue` |
| Seen | A place where secrets never live | Stop the action that would carry the value further (a commit, a send, caching, a hand-off, an output). Main agent: tell the developer now where the value lies and how to remove it, and propose rotation. Subagent: report the incident to the initiator | `incident` |
| About to emit | A place where secrets never live | Do not emit. Name where the value lies. No rotation proposal — nothing leaked | `refused` |
| About to emit | Another place (for example, an ignored file outside history) | Write the value there without a repeat of it in any message | `continue` |
| Pasted into the chat by the developer | — | Copy it nowhere. Tell the developer that the value is now in the transcript | `pasted` |

A message of the agent is always `agent-message`: a secret value never appears in a reply.

## Proportionality block

The section `## Proportionality` of the task file ([template](../templates/task_context.md)). Any contributor appends records, tagged with its id, at the end of the section. The developer's answer is a separate appended record. No record is ever rewritten. A subagent that does not write the task file carries its records in its report.

```
- row `<kind>` `<ref>` #<occurrence> — <text> — [agent-id] YYYY-MM-DD
- answer `<kind>` `<ref>` #<occurrence> → `<answer>` — [agent-id] YYYY-MM-DD
- created <what> — <where> — [agent-id] YYYY-MM-DD
```

- **Row** — one relaxation. `ref` identifies the subject inside its kind (a risk ID, a field name, a module and a feature). `text` is one logical line and never holds a secret value.
- **Answer** — the developer's answer to exactly one row.
- **Created** — an entity or data the task created; `where` is a system or a path.

A row is **open** until an answer with the same `kind`, `ref`, and occurrence exists.

**Row kinds** — closed list. A kind joins it only together with its allowed answers and its owner.

| Kind | Allowed answers | Owner |
|------|-----------------|-------|
| `unrun-tests` | `run-full-local` · `run-full-ci` · `skip` | test selection |
| `test-recipe-field` | `confirm` · `keep-default` | test selection |
| `risk-awaiting` | `handle` · `accept` | risk triage |
| `hardening` | `build` · `decline` | security exposure |

**Append a row** (the owner discipline does it):

1. `kind` is not in the list → write nothing: `KIND_UNKNOWN`, a defect of the caller.
2. `text` holds a secret value → write nothing; write the text again without the value: `SECRET_IN_TEXT`.
3. Re-read the task file.
4. The latest row of the same `kind` and `ref` is open → append nothing (`exists`). An open subject is never duplicated.
5. Otherwise append a row with the next occurrence: `#1` for a new subject. A subject that comes back after an answer (a risk whose trigger fired, a new set of unrun tests) gets a new open row, `#n+1`.

**Record a created entity** right after the task creates it (a customer on a test stand, a record in a test system):

1. `what` or `where` holds a secret value → write nothing: `SECRET_IN_TEXT`.
2. Re-read the task file and append the record.

**Show the block** at the design sign-off, at the commit sign-off, in the final report under `Autonomy: full`, and in the [audit](../phases/audit.md) report for the open rows of a closed task:

- No open rows → show nothing.
- Show one fragment in the `chat` register ([Output Styles](output-styles.md)); each line is self-contained. Per kind: the newest open rows first, up to `render_max` = 10, then one line `N more — the Proportionality section of the task file`.
- Final report: the fragment only. No questions, nothing goes to an owner.
- Audit report: one fragment per closed task; answers apply as at a sign-off.
- For each answer of the developer:
  - It matches no open row → `ANSWER_UNMATCHED`: ask again, append nothing.
  - It is not an allowed answer for the kind of its row → `ANSWER_NOT_ALLOWED`: ask again, append nothing.
  - Otherwise append the answer record, then give the owner the kind, the ref, the occurrence, the answer, the stop, and the date and task of the answer.
- A row without an answer stays open. Silence is never consent.

**Archive.** A size-triggered archive cycle of a live task file leaves this section whole. Answered pairs (a row and its answer) and created records leave only once the task is closed: moved by [audit](../phases/audit.md) Compact, or together with the whole task file. Open rows are never archived: a task file that holds one is never moved whole.

## Autonomy

- Under `Autonomy: full` the developer checkpoints are skipped, as [SKILL.md → Developer Checkpoints](../SKILL.md#developer-checkpoints) states. The block goes into the final report, without questions.
- An action with a hard-floor consequence stops the work even then: `ask-now` does not depend on `Autonomy`.
- `resolved (delegated)` never closes a hard-floor question.
- A confirmation that another discipline requires (a field of a test recipe, an access target) is not asked under `full`. Until it exists, the strict default applies.

## Boundaries

- No criterion of `## Validation Gates` changes. The developer checkpoints stay as they are; each one shows the block.
- The [Adaptation Profile](adaptation-profile.md) rule "secrets are never written" does not change.
- The [Safe Testing Principle](../phases/verify.md#safe-testing-principle) of verify stays in force; its list of destructive operations is read together with the hard floor.

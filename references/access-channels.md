# Access Channels — Stable Sanctioned Access for Agents

An **access channel** is a stable path to one resource that needs a credential or a privilege: a host, a cluster, a cloud account, a service, a database, the local machine under root. The developer sanctions a channel once, at the design sign-off of the concept that needs it. After that, the main agent and the subagents in its roles run the channel's operations through the privileged runner — a production release excepted — ([Proportionate Caution → Privileged commands](proportionate-caution.md#privileged-commands)) without a confirmation per call. Re-authorization happens only when the resource asks for it. The safety net is the channel journal and the channel report. The skill ships no runner, manifest, or helper: you write them in the project on demand.

## Contents

- [Channel](#channel) — the fields of a channel and the manifest
- [Operations](#operations) — patterns, the pinned endpoint, placeholders, consequence classes
- [Credential sources](#credential-sources) — where the runner takes the access from, in order of preference
- [The runner in channel mode](#the-runner-in-channel-mode) — what the runner checks before it runs an operation; when `confirm` still runs
- [Sanction](#sanction) — the channel card, the answer, hashes, widening and narrowing, autonomy
- [Re-authorization](#re-authorization) — only when the resource rejects the access
- [Roles](#roles) — which agent may call which channel
- [Journal and report](#journal-and-report) — the hash-chained journal; the report beside the proportionality block
- [Suspension and revocation](#suspension-and-revocation) — what stops a channel and how it comes back

## Channel

A channel lives in the manifest `.dev_flow/access/channels.yaml`: committed, no secret ever (an `access-manifest` place where secrets never live). The runner reads it at the moment of each call. Write it only through the steps of this reference; experience capture never writes it.

| Field | Holds |
|-------|-------|
| `id` | unique; `[a-z0-9-]+` |
| `resource` | `system`, `account`, `endpoint`, `marks` (`test` / `production` / `real-data`, can be empty) |
| `resource.endpoint` | the value, not a name: the server URL with the CA hash, the host with the host-key fingerprint, the cloud account or project ID |
| `source` | a [credential source](#credential-sources) with `reauth_command` and `identity` |
| `operations` | a non-empty list of [operations](#operations); empty only when `revoked` |
| `roles` | non-empty; `main` is the main agent; other names are subagent roles; `main` is never a subagent role |
| `probe` | the `id` of a `read` operation of this channel: the runner checks with it that the access works |
| `state` | `proposed` → `sanctioned` → `suspended` / `revoked` |
| `sanction` | the [sanction](#sanction) record; empty only in `proposed` |
| `pending` | widenings that wait for a sanction; never called |
| `description` | free text; no permission |

**Permission fields:** `resource`, `source`, `operations`, `roles`, `probe`. The other fields carry no permission. Every hash of permission fields is over their canonical form: keys sorted, no comments.

The journal `.dev_flow/access/journal.log` lies beside the manifest and is git-ignored (verify by effect). A stored secret never lies in the project tree ([Credential sources](#credential-sources)).

## Operations

An operation is an `argv` pattern in the form of the runner: anchored, fixed length, one element per position.

- **Pinned endpoint.** The pattern holds the endpoint as fixed elements: `--context` together with `--kubeconfig`, `--profile`, `--project`, the `ssh` host. A pattern without a pinned endpoint is invalid.
- **Placeholder.** A placeholder (`<…>`) stands only in an identifier position (a pod name, an image path, a tag) and has a declared `charset`. Never in the position of a verb, a resource kind, an option, or a query text. Its value never starts with `-`.
- **Delete verbs.** A delete verb (`delete`, `rm`, `drop`, `truncate`, `destroy`, `purge`) occurs only in a `destroy-test`, `named-hard-floor`, or `routine` operation with a declared device target.
- **Consequence.** The consequence of an operation is fixed once, at the sanction, from its class and the resource marks. At a call the runner checks only the placeholder values and the targets; it runs no semantic classifier.

| Class | Allows | Limit |
|-------|--------|-------|
| `read` | a read without a state change | `personal_data: none` or `present` is mandatory; `present` only as `named-hard-floor` (`real-data`) |
| `write` | a state change of a test resource | only a resource with the `test` mark; no delete verb |
| `destroy-test` | a deletion of test entities | only a resource with the `test` mark; only an entity that the runner derives from `argv` and that has a test marker or is a created record of the current task; no entity (truncate, bulk delete) — not `destroy-test` |
| `routine` | the privileged routine of the project (flash, system image, device debugging) on a fixed target | only a resource without the `production` or `real-data` mark; with elevation, only a root-owned helper; never a program that runs code or a config from a path that you can write; the write to the device target is sanctioned together with the operation |
| `named-hard-floor` | one operation with a hard-floor consequence: `production-write`, `irreversible-destruction`, `real-data`, a widening of access on the resource | an exact pattern with no target placeholder; its own answer from the developer; role `main` only; never `money` or `outward-message` |

**Content of a `named-hard-floor` operation.** When the operation carries content to a production resource (a chart, manifests, an image, migrations), it has `content: per-run`: each run goes through the `confirm` step of the runner. The dialog shows the command, HEAD, `git status` (uncommitted and untracked files), the diff summary since the HEAD of the last confirmed run (the whole tree on the first run), and the artifact digest when the operation pushes or deploys an image. The developer confirms the release there; under `Autonomy: full` only this line of work waits. A resource from which content reaches production — production reads from it (an image registry, a chart repository) or it deploys into production (a CI/CD pipeline trigger, a GitOps repository) — has the `production` mark: a write to it is a `named-hard-floor` operation with `content: per-run` too. On a resource with the `production` or `real-data` mark, every operation except `read` with `personal_data: none` is `named-hard-floor`. Production references an artifact by digest.

One answer covers repeat runs of the same operation; a `content: per-run` operation still confirms each run. A failed run is never retried automatically: stop this line of work and put it in the report. A rollback runs only when it is a named operation of its own.

A call that matches no operation is out of channel: it goes the path of the runner with `confirm`, or you propose a new operation ([Sanction](#sanction)). Never pick the nearest operation.

## Credential sources

Use the first one that the resource allows. The developer's storage preference in the [profile](adaptation-profile.md) wins over the tools at hand.

| Kind | Where the access is |
|------|---------------------|
| `native-auth` | the tool keeps it: `ssh` keys and agent, a credential helper, a cloud CLI session, a `kubeconfig` token the tool refreshes |
| `keyring` | the OS keyring; the developer enters the value by an [input route](proportionate-caution.md#secret-input); the runner reads it at the call |
| `masked-store` | outside the project tree, in `$XDG_DATA_HOME/dev-flow/<project-slug>/` with mode `0700`; only when the keyring is not available and the developer chose this |
| `root-helper` | a root-owned helper per operation in a directory that you cannot write (`/usr/local/sbin/`), its target and input path fixed inside, and one `sudoers` `NOPASSWD` line for it with the absolute path and exact arguments or none; the developer installs both; no `sudo` password is stored anywhere |

Each source also has these permission fields: `reauth_command` — the exact command that the developer runs to renew the access; `identity` — the account that the probe expects. A secret is stored only for a channel that the developer sanctioned. Never read the store and never call its restore command yourself. A setup or a renewal is done when the probe passes, not when a value is written.

## The runner in channel mode

Before the call, check the integrity of the runner yourself: root-owned, or its `sha256` equals the latest runner answer record ([Privileged commands](proportionate-caution.md#privileged-commands)). A mismatch: no call, tell the developer. The runner never checks itself. That record is the only runner integrity record for every channel: a runner change that the developer answered suspends no channel.

The runner takes `argv`, `target`, `reason`, `impact`, `role`, and `task`. In order; the first that fails ends the run:

1. **Role** — empty, or neither `main` nor a role of the skill or the project: refused.
2. **Match** — the first operation of a `sanctioned` channel (not `pending`) whose pattern matches `argv` fully. No match: `out-of-channel` — not a boundary refusal.
3. **Accepted state** — the hash of the permission fields equals the hash of the last accepted state (the sanction snapshot, moved by each journaled narrowing). Else: a removal of whole operations, targets, or roles is a narrowing — journal it with the new hash; anything else suspends the channel.
4. **Charset** — every placeholder value matches its `charset`.
5. **Role of the channel** — `role` is in `roles` (for `named-hard-floor`: `main`).
6. **Endpoint** — with a cleared environment (only the variables the tool needs), the tool itself resolves the endpoint (`ssh -G`, `kubectl config view --minify --kubeconfig=<pinned>`); it equals `resource.endpoint`. For a cloud CLI, the identity of the runner session equals `source.identity`.
7. **Target** — the targets derived from `argv` equal `target` and are in the targets of the operation.
8. **Entity** — for `destroy-test`, the entity derived from `argv` passes its rule; a failure is refused but is not a boundary refusal.
9. **`check`** of the runner, without exceptions; the destruction check skips only the targets of a `routine` operation.
10. **Content** — for `content: per-run`, run the `confirm` step of the runner with what the dialog shows ([Operations](#operations)); `declined` ends the run.
11. **Execute** — in the cleared environment of step 6; `confirm` is skipped, except at step 10. The credential goes through stdin, a file descriptor, or a child environment variable, never `argv`. The output is masked for the secret and its encodings; no verbose mode. For an operation that runs project code (tests against a test resource — only a `test` resource), check the files the run wrote for the secret value: a match is an incident and suspends the channel.
12. **Authentication rejected** — when the output carries an authentication-rejection sign of the tool and this call is not the probe, run the probe directly; when it fails the same way, return `reauth-required`.
13. **Journal** the result.

A refusal at steps 4, 6, 7, or 9 is a **boundary refusal**. The runner counts boundary refusals per channel, not per task: the second within 24 hours suspends the channel.

## Sanction

**The card.** When a concept needs a resource, put a channel card into the concept's `Access Channels` subsection ([concept template](../templates/concept.md)): the resource with its endpoint and marks, the source with `reauth_command` and `identity`, each operation with its class and targets, the roles, the probe. Add one Design Decisions record (`proposed`) for the card and one for each `named-hard-floor` operation ([Interview Mode](interview-mode.md)). The design sign-off shows them like any decision; implementation does not start on them while they are `proposed`. Build each channel narrow: drop an operation that names a shell, an interpreter, a wrapper, an editor, a pager, or code from a path that you can write, and say why.

**The answer.** A sanction is an explicit yes to this card — `Autonomy: full` is not an answer to it. A `named-hard-floor` operation without its own yes leaves the channel. A no to the card removes the card. The sanction record holds the quote, the date, the stop, the hash of the permission fields, the snapshot of the permission fields, the answers per `named-hard-floor` operation. When the runner is root-owned, give the developer the command that installs the snapshot beside it. With a `root-helper` source not yet installed, the channel stays `proposed`; give the developer the exact lines. After the sanction, run the probe and check `identity`.

**Outside the design sign-off.** A need that the concept did not foresee (an autonomous run reached an unknown resource) is a separate question to the developer. This line of work waits; the rest of the work goes on; the need goes into the report. Never reach the resource another way. The sanction is `access-relaxation` on the hard floor: it needs the developer's word under any `Autonomy`.

**Change.** A field without permission: no sanction. A widening (a new operation, target, role, a wider pattern, another source) goes into `pending` and needs a new sanction; the rest of the channel keeps working. A narrowing is only a removal of whole operations, targets, or roles; any edit inside an operation is a widening. A sanction never expires.

## Re-authorization

Only the resource starts it. On `reauth-required`:

- never retry with other credentials, never switch the source, never look for another way in;
- stop only this line of work; the rest goes on;
- ask the developer to run the channel's `reauth_command` (for example `aws sso login --profile staging`, `ssh-add`) by an [input route](proportionate-caution.md#secret-input); never compose a login command at that moment; in an autonomous run, put the need into the final report;
- after the developer acts, run the probe; it checks that the authenticated account equals `identity`; then continue.

The sanction does not change. A subagent hands `reauth-required` to the initiator.

## Roles

A channel lists the roles that may call it. `main` is never a subagent role: a brief never sets it, a subagent never passes it. A subagent in a listed role calls the runner with its role; any other subagent hands the need to the initiator. The initiator's brief sets the role of a subagent. A `named-hard-floor` operation is for role `main` only: the hard-floor rules of every other role stay in force without exception. A role is a guard against drift, not authentication.

## Journal and report

The runner appends one journal line per call and per manifest change: time, role, task, channel, operation, the target derived from `argv`, the result (`ok`, `exit:<code>`, `refused:<reason>`, `declined`, `reauth-required`, `suspended`, `manifest-change:<sanction | narrow | suspend | revoke | runner>`), HEAD for a run of a `content: per-run` operation, and the `sha256` of the previous line. No secret, no command output, no `argv` beyond the operation pattern. `audit` checks the chain and compares the manifest history in git with the sanctions: an edit with a recomputed hash shows there.

**The channel report** is part of showing the [proportionality block](proportionate-caution.md#proportionality-block): wherever a stop or a final report shows the block, it shows the report too. Per task or autonomous run:

| Section | Holds |
|---------|-------|
| usage | per channel, the count of calls per class |
| hard floor | each `named-hard-floor` run on its own line: time, role, operation, target, HEAD, result |
| manifest changes | sanctions, narrowings, suspensions, revocations, runner changes |
| refusals | refusals of the runner with the reason |
| re-authorization | `reauth-required`: channel, time, whether renewed |
| suspensions | each suspension with the cause |
| needs | new channels or operations that wait for the developer |
| chain | the count of journal lines and the hash of the last one |

An empty report is not shown.

## Suspension and revocation

A channel becomes `suspended` when:

- the permission fields differ from the last accepted state, and the change is not a narrowing;
- the secret of its source is seen in a place where secrets never live, or in a file that a run wrote — an incident; propose a rotation;
- the second boundary refusal for the channel within 24 hours.

A suspended channel runs no operation; its calls go the path of the runner with `confirm`. Only a new sanction restores it. Only the developer revokes a channel; a revoked channel stays in the manifest as history, with no operations. No channel expires with time.

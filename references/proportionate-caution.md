# Proportionate Caution — The Hard Floor, Secrets, and the Proportionality Block

Cross-cutting discipline of **every phase**. Not a pipeline stage, no command. Size caution to the consequence of an action, not to the action. The **hard floor** is the closed set of consequences that always need the developer's word now; no autonomy mode or unsanctioned declaration relaxes it — only a `named-hard-floor` operation of a sanctioned access channel carries that word in advance, for role `main`. A seen secret is an incident only in a place where secrets never live, and the agent never puts a secret value there. A secret that the agent needs comes from the developer by an input route outside the chat. A privileged or remote command runs only through the privileged runner of the project. A relaxation of a listed row kind leaves a row in the **proportionality block**, which the developer checkpoints show.

## Contents

- [Hard floor](#hard-floor) — the consequences that need the developer's word now, what is not a consequence, the unestablished fact, what the agent never settles alone
- [Asking now](#asking-now) — `proceed` / `relay` / `ask-now` per action and caller
- [Secrets](#secrets) — what a secret is; the closed list of places where secrets never live
- [Seen and emitted secrets](#seen-and-emitted-secrets) — the reaction per event and place: continue, incident, refused, pasted
- [Secret input](#secret-input) — a needed secret never asked for in the chat; the input routes in order of preference; subagent relay
- [Privileged commands](#privileged-commands) — the privileged class; the runner, its boundary and steps; adequacy; sanctioned channels; projects with constant privileged work
- [Proportionality block](#proportionality-block) — the task-file section: record forms, row kinds, append, created records, showing, archive
- [Autonomy](#autonomy) — what `Autonomy: full` skips and what still stops the work, secret input, the runner `confirm`, and channel sanctions included
- [Boundaries](#boundaries) — what this discipline does not change

## Hard floor

An action with any of these consequences needs the **developer's word now** — not a row in the block, not a delegated decision; a `named-hard-floor` operation of a sanctioned [access channel](access-channels.md#operations), role `main`, carries it in advance. Classify from facts: which system, which data, whether a recovery exists.

| Consequence | The action has it when |
|-------------|------------------------|
| `real-data` | It writes, changes, deletes, or exports (out of the system, also into the agent's own output) data of real people — in an external system or copied from one to the local machine. Reading aggregates that hold no data of people does not count |
| `production-write` | It writes to production |
| `money` | It triggers a payment, a subscription, or billing — also for an entity the task created |
| `outward-message` | It sends an email, an SMS, a push notification, or a chat message to a real person |
| `irreversible-destruction` | It discards content that cannot be recovered: a reset of uncommitted changes (`reset --hard`, `checkout --`, `restore`, `stash drop`), or a file removal or a replacement of the whole content, when the current task did not write that content and vcs history does not hold it. Or it deletes data outside a confirmed test target, and that data cannot be recovered |
| `access-relaxation` | Without the developer's confirmation, it removes or bypasses a command gate, widens the permissions of the agent runtime, sanctions or widens an [access channel](access-channels.md#sanction), or creates or changes the privileged runner ([Privileged commands](#privileged-commands)). A `proposed` channel entry and a narrowing are not a relaxation |

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

A `named-hard-floor` operation of a sanctioned [access channel](access-channels.md#operations), called by role `main`, carries the developer's word given in advance for that one operation: `proceed`.

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
| Pasted into the chat by the developer | — | Copy it nowhere. Tell the developer that the value is now in the transcript, and propose an input route for the next entry ([Secret input](#secret-input)) | `pasted` |

A message of the agent is always `agent-message`: a secret value never appears in a reply.

## Secret input

When an action needs a secret that you do not have — a `sudo` password, a tracker login, a test-account password, a service token — never ask for the value in a message and never put it in a command line that you compose. Propose an input route: the value goes from the developer straight to the consuming process; read only the exit code and the output without the value.

Take the first route whose condition holds:

| Route | Condition | Means |
|-------|-----------|-------|
| `native-auth` | The consumer authenticates without a typed value | ssh keys and `ssh-agent`, a credential helper, the OS keyring, an existing session, a `0600` file that the tool itself reads |
| `hidden-dialog` | The consumer has its own askpass hook, and a dialog tool is available in the current graphical session | `SSH_ASKPASS`, `GIT_ASKPASS`; for a privileged command, `SUDO_ASKPASS` / `pkexec` only inside the runner ([Privileged commands](#privileged-commands)) |
| `wrapper` | The consumer reads the value from stdin, a file descriptor, or a child-process environment variable, and a dialog tool is available in the current graphical session | A wrapper in the project that you write on demand: it asks for the value with a graphical hidden-input dialog tool (`kdialog --password`, `zenity --password`, a GUI `pinentry` flavour such as `pinentry-qt` / `pinentry-gnome3` / `pinentry-gtk-2`, `osascript` `display dialog … with hidden answer` on macOS, PowerShell `Get-Credential` on Windows) and sends the value that tool returns only to the consumer — parsed out in memory where the tool wraps it (pinentry `D` lines, osascript `text returned:`, the credential object); it never prints, stores, or contains the value |
| `developer-terminal` | No condition above holds — no graphical session, no dialog tool, or the consumer takes the value only as an argument | Give the exact command; the developer runs it in their own terminal with the hidden prompt of the tool or of a wrapper (`read -s`); read the result. A consumer that takes the value only as an argument: the developer reads it with `read -s` into a variable and passes the variable, then runs `unset` on it; the developer never types the value on the command line; tell the developer that the value stays visible in the process list while the tool runs |

- **Graphical session** — a fact of the current session: check it each time, in the agent's own shell (a GUI tool needs the session's display variables there, not only on the desktop) — `$DISPLAY` / `$WAYLAND_DISPLAY` on Linux; a logged-in desktop session not reached over SSH on macOS (`launchctl managername` reports `Aqua`) and on Windows; never record it.
- **Dialog tool** — a machine fact: read it from `environment.md` of the [Adaptation Profile](adaptation-profile.md), else check it and record it per that file. A tool counts only if it draws its own window from the agent's shell, which has no terminal: `systemd-ask-password` and a terminal `pinentry` (curses, tty) need a TTY or a password agent and hang or fail there, so they do not count.
- **No echo** — run the route without verbose or trace flags (`-v`, `--debug`, `set -x`) that can print the value.
- **Wrapper** — a project tool, never a skill file. A one-time wrapper stays in the session workspace (`/tmp/{project-slug}/`): create the directory with mode `0700`; before you write or run the wrapper, check that you own it and that its mode is `0700`. A reused wrapper goes among the project tools with the developer's knowledge.
- **Reuse** — to reuse a value, let the tool keep it in its own store (credential helper, keyring); store it nowhere else.
- **Subagent** — use `native-auth` only. For any other route, hand the need to the initiator without the value; never ask the initiator for the value. The initiator runs the route and the action itself, or runs the subagent again when the own store of the tool holds the value.
- **Precedence** — a tracker integration or another skill that asks for a password in the chat yields to this route.
- **`Autonomy: full`** — the same route; the work waits for the developer's input.
- **Pasted anyway** — `pasted` above, and propose a route for the next entry.

## Privileged commands

Never run a command of the privileged class directly — also not as a retry after `permission denied`. Run it only through the **privileged runner** of the project, `.dev_flow/tools/run-privileged`.

**The privileged class** — closed list of signs:

| Sign | The command has it when |
|------|-------------------------|
| `elevation` | The program is `sudo`, `doas`, `su`, `pkexec`, `run0`, `runuser`, or `machinectl shell`; `systemctl` without `--user` with any verb except `status`, `show`, `cat`, `list-*`, `is-*`, `get-default`, `help`; a state change that polkit or the system bus authorizes (`loginctl`, `udisksctl`, `nmcli` connection or device changes, `hostnamectl` / `timedatectl` / `localectl` `set-*`, `busctl call` on the system bus); `mount` / `umount` |
| `remote-access` | `ssh` / `mosh` to a host; `scp` / `sftp` / `rsync` with a remote side; `kubectl` / `helm` / `oc`; `aws` / `gcloud` / `az` / `doctl`; `terraform` / `pulumi` apply / destroy / import; `ansible`; a database client to a host that is not `localhost` / loopback, or to an unknown host; `systemctl -H` / `-M` |

Not in the class: git transport (`fetch`, `pull`, `push`, `clone`); a package manager that fetches project dependencies; a read of a public HTTP resource; local containers; commands to `localhost` / loopback — this exclusion wins over the `remote-access` list, decided from what `ssh -G <host>` resolves (hostname, user, `ProxyJump`, `ProxyCommand`), not from the command text; it does not apply to a login as another user (a user other than the current one by `user@`, `-l`, `-o User`, or `ssh` config; any `root`) or through a proxy and a loopback port that forwards to a remote host (an `ssh -L` tunnel, `kubectl port-forward`), which counts as that host; a subcommand with no remote endpoint (`helm template` / `lint`, `kubectl kustomize`, `kubectl config view`, `--dry-run=client`).

**The runner** — a project tool that you write on demand, never a skill file. It holds no secret. Its `boundary` block holds the allowed hosts and contexts, the path prefixes for privileged writes, and the allowed `ssh` `-o` options. The operations that run without `confirm` are not in the runner: they live in the manifest of the [access channels](access-channels.md#operations).

A pattern is anchored and has a fixed length: one element per position; a placeholder (`<…>`) covers one element, and the value it matches never starts with `-`.

To create the runner or change it, `boundary` included, is `access-relaxation` ([hard floor](#hard-floor)). Propose that the developer installs it root-owned in a directory that you cannot write; else record its `sha256` in the answer record and compare it before each call with the latest runner answer record, archived ones included — a mismatch: do not call it, tell the developer; no record: ask the developer and record the answer.

The runner takes `argv` (an argument array, not a shell string), `target`, `reason` (why the task needs it), and `impact` (what changes, where, whether a recovery exists). Steps, in order; the first that fails ends the run:

| Step | Does | On failure |
|------|------|------------|
| `check` | Derives every target from `argv` (host, context, device, write path) and refuses a mismatch with `target`. Refuses: a shell or a session under elevation (`sudo -s` / `-i`, `su` without `-c`, `ssh` without a command — `-N` with port forwarding excepted); as the privileged program, a shell, an interpreter, a wrapper (`env`, `xargs`, `find -exec`, `nice`, `timeout`, `nohup`, `chroot`, `systemd-run`, `script`), an editor, or a pager — an edit goes through `sudoedit`; in every argument that a remote shell parses (the `ssh` / `mosh` remote command, a remote path of `scp` / `rsync`, `rsync -e` / `--rsh` / `--rsync-path`), a shell metacharacter (`` ; & \| $ ` < > ( ) * ? ~ { } ' " \ # ``, whitespace inside an element, or a newline); an `ssh` `-o` / `-F` option outside `boundary`; a secret in `argv`; a target outside `boundary`; access widening (a write path to `sudoers`, PAM, `/etc/group`, `/etc/passwd`, `/etc/shadow`; a user added to a group; recursive `chmod` / `chown` outside `boundary`); destruction outside `boundary` (`rm -r` of the root or a system path, `dd` to a device, `mkfs`, `flashrom`) — not refused for a target of the matching `routine` operation of a sanctioned channel; a change of the runner | `refused: <reason>` |
| `confirm` | Shows `argv`, `target`, `reason`, `impact`, and the `check` result in a dialog of the current graphical session — on `/dev/tty` only when the developer runs the runner; never from stdin; the developer confirms or declines there, in the same place as the password. Out of channel, every time, also under `NOPASSWD`. Skipped only for a call that matches an operation of a sanctioned channel in an allowed role, not one with `content: per-run` ([Access Channels](access-channels.md#the-runner-in-channel-mode)) | `declined` |
| `execute` | `elevation`: `sudo -A -k` with `SUDO_ASKPASS`, or `pkexec`; no verbose or trace flags. `remote-access`: `native-auth` or askpass ([Secret input](#secret-input)); each element of the `ssh` remote command is quoted for the remote shell separately (`printf %q`) | exit code |

The runner calls `sudo`, `ssh`, and the dialog tool by absolute path; a Python runner runs with `-I`.

Read only the result: `refused`, `declined`, or the exit code and the output.

- **Adequacy** — before you call the runner, make sure that the command is necessary for the task Intent, that it is the narrowest one (restart one unit, not reboot the machine), and that `impact` is true and complete. Else narrow the command or complete `impact`.
- **Hard floor** — a consequence on the hard floor (for example, `production-write`) is still asked now in the chat, and the runner then confirms the exact command — except a `named-hard-floor` operation of a sanctioned channel, which is not asked; the runner confirms it only when it carries content (`content: per-run`). A `read` operation with `personal_data: present` exists only as `named-hard-floor`.
- **Never feed `confirm`** — no pipe, no pty, no input automation.
- **No graphical session** — give the developer the runner command for their own terminal.
- **No runner** — propose to write it (the developer's word now), or give the developer the command for their own terminal. A written runner gets an entry in `.dev_flow/tools/_index.yaml`: `regime: developer-word`, `self_test: null` ([Experience Capture](experience-capture.md#applying-experience--auto-apply-through-structural-gates)).
- **Subagent** — call the runner only for an operation of a channel that lists your role, and pass the role (never `main`). Else hand `argv`, `target`, `reason`, and `impact` to the initiator.
- **`Autonomy: full`** — `confirm` still waits for the developer; an operation of a sanctioned channel does not, except one with `content: per-run`.
- **Never** change `sudoers`, groups, or udev rules yourself — give the exact lines to the developer.

**Projects with constant privileged work** (firmware, debugging on real devices, system images):

- First remove the elevation. When the privilege is needed only for device access (a programmer, a serial port, a USB drive), propose a one-time udev rule or group (`dialout`, `plugdev`); the developer runs the lines. After that the command (`openocd`, `esptool`, `dfu-util`) is outside the class and runs directly.
- What stays privileged becomes a `routine` operation of a channel to the local machine with a `root-helper` source. Propose a root-owned helper per routine operation in a directory that you cannot write (`/usr/local/sbin/`), with its target and input path fixed inside, and one `sudoers` `NOPASSWD` line for it: the absolute path of the helper, exact arguments or none, never a wildcard. Never a `NOPASSWD` line for `dd`, `cp`, `tee`, `mount`, an editor, a pager, a shell, an interpreter, `env`, or `find`. The developer installs the helper and the line; the operation names the same helper.
- Propose this setup, as a channel card, when the project concept or `_framework.md` describes work with devices or system images, or when a privileged command of the same shape recurs in the task.

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

- No open rows → show nothing of the block. The [channel report](access-channels.md#journal-and-report) goes with it whenever the report is not empty.
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
- An action with a hard-floor consequence stops the work even then: `ask-now` does not depend on `Autonomy`. A `named-hard-floor` operation of a sanctioned channel is the exception: its answer was given in advance; one that carries content still waits for its `confirm`.
- A needed secret waits for the developer's input even then ([Secret input](#secret-input)).
- The `confirm` step of the privileged runner waits for the developer even then ([Privileged commands](#privileged-commands)); an operation of a sanctioned channel does not need it, except one with `content: per-run`.
- A channel sanction and a `reauth-required` wait for the developer even then; only that line of work waits ([Access Channels](access-channels.md#sanction)).
- `resolved (delegated)` and `resolved (agent)` never close a hard-floor question.
- A confirmation that another discipline requires (a field of a test recipe) is not asked under `full`. Until it exists, the strict default applies. A channel sanction is not such a confirmation.

## Boundaries

- No criterion of `## Validation Gates` changes. The developer checkpoints stay as they are; each one shows the block.
- The [Adaptation Profile](adaptation-profile.md) rule "secrets are never written" does not change.
- The [Safe Testing Principle](../phases/verify.md#safe-testing-principle) of verify stays in force; its list of destructive operations is read together with the hard floor.

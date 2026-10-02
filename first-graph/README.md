# Prepare your first Graph

> For agents: use only the commands, flags and fields on this page, exactly as written. Run `interf --version` first; if it prints a different version, use `interf --help` and `interf <command> --help` instead of this page. llms.txt at the repository root lists every page.
>
> Checked against `@interf/compiler` 0.51.0.

This tutorial prepares a Graph from four public FOMC statements with your own
agent, then hands the Graph to another agent and catches one changed file. Your
own agent needs no Interf account and no credits; its provider bills its work as
usual.

<!-- capture:header -->
Captured on 2026-10-02 with `@interf/compiler` 0.51.0 and Claude Code 2.1.287. Your ids, digests, times and costs will differ.
<!-- /capture:header -->

## What you need

- An Apple silicon Mac with Node.js 22.19 or later.
- `@interf/compiler` 0.51.0: `npm install --global @interf/compiler`. Without
  installing, type `npx @interf/compiler` wherever this page says `interf`.
- Claude Code, installed and signed in. Codex works the same way with
  `"type":"codex"`.

## The folder

[`fed-watch/fomc/`](fed-watch/fomc/) holds four FOMC statements from 2024: July
31, September 18, November 7 and December 18. They are copied from the website of
the Board of Governors of the Federal Reserve System, which publishes its
information in the public domain unless otherwise indicated. Each file adds a
title and a source line naming the Board above the statement's text. Two of the statements record a dissent.

Copy the folder into your home folder, so every command below can name it as
`$HOME/fed-watch`:

```sh
git clone --depth 1 https://github.com/interf-labs/compiler.git interf-compiler
cp -R interf-compiler/first-graph/fed-watch "$HOME/fed-watch"
```

You will use the [Fed watch](../context-interfaces/examples/fed-watch/README.md)
Context Interface. It declares what the Graph must contain: one policy path
across the meetings, and the meetings, decisions, assessments, guidance and
dissents it links. Check it offline:

```sh
interf context-interface validate --file interf-compiler/context-interfaces/examples/fed-watch/context-interface.json
```

<!-- capture:validate -->
```text
$ interf context-interface validate --file interf-compiler/context-interfaces/examples/fed-watch/context-interface.json
Valid Context Interface: sha256:7671c166d970b176d373566adf9e3818da9d0404092538d2254d5d02e00b160d
```
<!-- /capture:validate -->

## 1. Start the Runtime and connect your agent

```sh
interf runtime start
```

From a folder where you already use Claude Code, write the connection request
with absolute paths and connect. The paths are where Claude Code starts and your
home folder; they hold no credentials.

```sh
printf '{"type":"claude-code","auth":{"mode":"cli","customer_context":{"project_root":"%s","home":"%s"}}}\n' "$PWD" "$HOME" > agent.json
interf agents connect --request agent.json
```

Note the `agent_id` it prints, then make that agent the default for the three
jobs this tutorial runs:

```sh
printf '{"agent_id":"%s"}\n' "<agent-id>" > choice.json
interf agents default source-scan --request choice.json
interf agents default build-plan-preparation --request choice.json
interf agents default graph-build --request choice.json
interf agents ls
```

`interf agents ls` shows each of those job defaults as `ready`.

## 2. Create the Graph and scan the folder

```sh
interf graphs create fed-watch --intent "Follow how the FOMC's decisions, assessments, guidance and dissents changed across its 2024 meetings."
interf sources add local-folder "$HOME/fed-watch"
interf graphs source-add fed-watch <source-id>
interf sources refresh <source-id>
interf runs status <run-id>
```

`sources add` prints the Source id. `sources refresh` starts a Scan in which your
agent reads the four files, read-only, and prints its Run id. Repeat
`runs status` until the Run has succeeded.

## 3. Load and approve the Context Interface

```sh
interf context-interface load fed-watch --file interf-compiler/context-interfaces/examples/fed-watch/context-interface.json
```

<!-- capture:load -->
```text
$ interf context-interface load fed-watch --file interf-compiler/context-interfaces/examples/fed-watch/context-interface.json
Recorded Context Interface candidate.
  Graph: fed-watch
  input digest: sha256:c1b5087e2e9c4042b32722e8e139639581fb4aeea26869924d405483a994bf12
  Interface digest: sha256:7671c166d970b176d373566adf9e3818da9d0404092538d2254d5d02e00b160d
  Consumption: source-linked
  Run with --json for the exact candidate and Interface document.
```
<!-- /capture:load -->

The digest matches the one `validate` printed. Review the exact document, then
approve it:

```sh
interf context-interface review fed-watch <input-digest> <interface-digest>
interf context-interface approve fed-watch <input-digest> <interface-digest>
```

## 4. Review the Build Plan and its cost

```sh
interf build-plan create fed-watch
interf runs status <run-id>
interf build-plan history fed-watch
```

<!-- capture:history -->
```text
$ interf build-plan history fed-watch
Build Plan history for fed-watch
  Graph state: 4
→ fed-watch-build-plan · unapproved · current Plan · 2026-10-02T04:28:47.035Z
    digest: sha256:53208b2bc3ada461b49bc6e0e3b8b75a1b0678bf2492015a511cbbf84d7f22b9
    Interface: interface-approval_5cd1bddd7d8c67a15665a6dc5d0287ab92c040ca9c6fb8de26dde8c355019dd0
    Build target: Not selected for Build
  Run with --json for exact saved version identities.
```
<!-- /capture:history -->

```sh
interf build-plan show fed-watch <build-plan-id> <plan-digest>
```

The review shows the ordered stages, the files each stage reads, the agent that
will build, and the estimated cost of building with your agent beside Interf
Agent:

<!-- capture:plan -->
```text
$ interf build-plan show fed-watch fed-watch-build-plan sha256:53208b2bc3ada461b49bc6e0e3b8b75a1b0678bf2492015a511cbbf84d7f22b9
Build Plan fed-watch-build-plan
  Graph: fed-watch
  digest: sha256:53208b2bc3ada461b49bc6e0e3b8b75a1b0678bf2492015a511cbbf84d7f22b9
  Task: Follow how the FOMC's decisions, assessments, guidance and dissents changed across its 2024 meetings.
  Sources: 1 (0 refresh-advisory)
    fed-watch: inventory-82db2dda76838d6c0c16e14402cc6174
  Stages: 1
    build-fomc-2024-meetings: Build the 2024 FOMC meeting graph and policy-path entrypoint · 4 file(s)
      Cost: LLM estimate unavailable because exact Agent pricing is unavailable. Execution charge: $0.00. Total estimate unavailable because at least one component is unavailable.
  Approval: ready
  Run with --json for the exact Plan inspection.
  Build Agent: agt-claude-code
  Cost: LLM estimate unavailable because exact Agent pricing is unavailable. Execution charge: $0.00. Total estimate unavailable because at least one component is unavailable.
  Estimate, not a quote. Same Plan and remaining work. Actual execution can vary.
  Projected tokens: Input: 0–110,000 · Output: 256–85,000 · Cache read: 0–1,290,000 · Cache write (5m): 0 · Cache write (1h): 0
  agt-claude-code (selected)
    Model: Model unavailable
    Model payer: Your Agent provider
    Execution location: Local
    LLM: LLM estimate unavailable because exact Agent pricing is unavailable.
    Execution: Execution charge: $0.00.
    Total: Total estimate unavailable because at least one component is unavailable.
    Customer charge: Billed by your Agent provider; subscription marginal dollars may be unknown.
    Rates: Exact rates unavailable.
    Rate evidence: Rate provenance unavailable.
  Interf Agent
    Model: vercel-ai-gateway/anthropic/claude-sonnet-4.6
    Model payer: Interf pays the model provider
    Execution location: Local
    LLM: Supplier LLM estimate: $0.00384–$2.00.
    Execution: Execution charge: $0.00.
    Total: Supplier Total estimate: $0.00384–$2.00.
    Customer charge: Customer charge unavailable: no credit quote for this work.
    Rates: Input: $3.00 · Output: $15.00 · Cache read: $0.30 · Cache write (5m): unavailable · Cache write (1h): unavailable per 1M tokens.
    Rate evidence: Snapshot 2026-09-11 · https://ai-gateway.vercel.sh/v1/models · ai-gateway.vercel.sh · standard
  Stage build-fomc-2024-meetings projected tokens: Input: 0–110,000 · Output: 256–85,000 · Cache read: 0–1,290,000 · Cache write (5m): 0 · Cache write (1h): 0
    agt-claude-code (selected): LLM estimate unavailable because exact Agent pricing is unavailable. Execution charge: $0.00. Total estimate unavailable because at least one component is unavailable. Billed by your Agent provider; subscription marginal dollars may be unknown.
    Interf Agent: Supplier LLM estimate: $0.00384–$2.00. Execution charge: $0.00. Supplier Total estimate: $0.00384–$2.00. Customer charge unavailable: no credit quote for this work.
  Interf Agent execution is unavailable. Viewing this comparison does not enable it. Local Interf model estimates are not customer credit quotes.
  reviewed pricing: eyJiaWxsaW5nX21vZGUiOiJ1bmtub3duIiwicmVhc29uIjoicHJvdmlkZXItZW5kcG9pbnQtdW5hdmFpbGFibGUiLCJzdGF0dXMiOiJ1bmF2YWlsYWJsZSJ9
```
<!-- /capture:plan -->

The cost is an estimate, not a quote; your numbers will differ.

## 5. Approve and build

Approve only the exact Plan you reviewed. Use the Graph state from
`build-plan history`, and the Build Agent and reviewed pricing that
`build-plan show` printed:

```sh
interf build-plan approve fed-watch <build-plan-id> <plan-digest> --expected-state-version <state> --agent-id <agent-id> --reviewed-pricing <receipt>
```

Approval prints the exact `interf build` command for this Plan, including the
Graph state it recorded. Run that command with `--wait` added. When the Build
succeeds it prints `Graph revision: <revision-id>`; inspect that Revision:

```sh
interf build fed-watch --idempotency-key <approval-id>-<approved-state> --approval-id <approval-id> --build-plan-id <build-plan-id> --plan-digest <plan-digest> --graph-state-version <approved-state> --agent-id <agent-id> --reviewed-pricing <receipt> --wait
interf graphs inspect <revision-id> --graph fed-watch
```

<!-- capture:inspect -->
```text
$ interf graphs inspect revision_dab3922a72ad39e8c75ca80cd0922fd19e2d388e0b51f397e6ed0d930b71fa9a --graph fed-watch

  revision_dab3922a72ad39e8c75ca80cd0922fd19e2d388e0b51f397e6ed0d930b71fa9a READY
    graph: fed-watch
    Source Inventories:
      fed-watch (src-200a559929c27071) · inventory-82db2dda76838d6c0c16e14402cc6174 · observed 2026-10-02T04:28:06.123Z
        scan again: interf sources refresh src-200a559929c27071
    proof · Source coverage: 4/4 accounted for
      missing: none
      blocked: none
    proof · Context Interface: satisfied
    files:
      used     src-200a559929c27071:fomc/2024-07-31-statement.md [build-fomc-2024-meetings] -> fed-watch.md (policy-path), meetings/2024-07-31/meeting.md (meeting-2024-07-31), meetings/2024-07-31/policy-decision.md (decision-2024-07-31), meetings/2024-07-31/economic-assessment.md (assessment-2024-07-31), meetings/2024-07-31/forward-guidance.md (guidance-2024-07-31)
      used     src-200a559929c27071:fomc/2024-09-18-statement.md [build-fomc-2024-meetings] -> fed-watch.md (policy-path), meetings/2024-09-18/meeting.md (meeting-2024-09-18), meetings/2024-09-18/policy-decision.md (decision-2024-09-18), meetings/2024-09-18/economic-assessment.md (assessment-2024-09-18), meetings/2024-09-18/forward-guidance.md (guidance-2024-09-18), meetings/2024-09-18/dissent-michelle-w-bowman.md (dissent-2024-09-18-michelle-w-bowman)
      used     src-200a559929c27071:fomc/2024-11-07-statement.md [build-fomc-2024-meetings] -> fed-watch.md (policy-path), meetings/2024-11-07/meeting.md (meeting-2024-11-07), meetings/2024-11-07/policy-decision.md (decision-2024-11-07), meetings/2024-11-07/economic-assessment.md (assessment-2024-11-07), meetings/2024-11-07/forward-guidance.md (guidance-2024-11-07)
      used     src-200a559929c27071:fomc/2024-12-18-statement.md [build-fomc-2024-meetings] -> fed-watch.md (policy-path), meetings/2024-12-18/meeting.md (meeting-2024-12-18), meetings/2024-12-18/policy-decision.md (decision-2024-12-18), meetings/2024-12-18/economic-assessment.md (assessment-2024-12-18), meetings/2024-12-18/forward-guidance.md (guidance-2024-12-18), meetings/2024-12-18/dissent-beth-m-hammack.md (dissent-2024-12-18-beth-m-hammack)
```
<!-- /capture:inspect -->

Source coverage counts every selected file as accounted for. It does not prove
that every fact was understood.

## 6. Hand the Graph to another agent

```sh
interf graphs archive <revision-id> --graph fed-watch --portable --output "$HOME/fed-watch-graph.zip"
mkdir "$HOME/fed-watch-graph"
unzip "$HOME/fed-watch-graph.zip" -d "$HOME/fed-watch-graph"
```

The folder holds `README.md`, `AGENTS.md`, `.gitignore`, `graph.tar.gz` and
`source-requirements.json`. Any agent can follow its `AGENTS.md` without
installing Interf. Create the private file that records where the cited Sources
are and what was checked:

```sh
interf graphs files "$HOME/fed-watch-graph" --init
```

<!-- capture:files-init -->
```text
$ interf graphs files "$HOME/fed-watch-graph" --init
Private file: $HOME/fed-watch-graph/.interf-local.json
Digest: sha256:68e3be2c978d7764f376e2d2715997e679876b51a154c58362bd59dfd902b15b
Reported checks; access and freshness are not guaranteed.
unchecked: fomc/2024-07-31-statement.md
unchecked: fomc/2024-09-18-statement.md
unchecked: fomc/2024-11-07-statement.md
unchecked: fomc/2024-12-18-statement.md
```
<!-- /capture:files-init -->

Map the cited Source to the folder that holds `fomc/`, using the digest that
`--init` printed. The locator must be an absolute path:

```sh
interf graphs files "$HOME/fed-watch-graph" --source <source-id> --target "{\"kind\":\"local-folder\",\"locator\":\"$HOME/fed-watch\"}" --expected-digest <digest>
```

## 7. Change one statement and check

```sh
echo "Edited after the Build." >> "$HOME/fed-watch/fomc/2024-09-18-statement.md"
interf graphs files "$HOME/fed-watch-graph" --check
```

<!-- capture:files-check -->
```text
$ interf graphs files "$HOME/fed-watch-graph" --check
Private file: $HOME/fed-watch-graph/.interf-local.json
Digest: sha256:29222d6860ae089429fdccf67717eeeb61cca1872be3d651c2edcc40f42580c2
Reported checks; access and freshness are not guaranteed.
exact: fomc/2024-07-31-statement.md (reported 2026-10-02T04:34:36.534Z)
changed: fomc/2024-09-18-statement.md (reported 2026-10-02T04:34:36.536Z)
exact: fomc/2024-11-07-statement.md (reported 2026-10-02T04:34:36.539Z)
exact: fomc/2024-12-18-statement.md (reported 2026-10-02T04:34:36.541Z)
```
<!-- /capture:files-check -->

The edited statement reports `changed`; the others report `exact`. These are
reported checks, not proof of access or freshness. An agent explains the change
before it relies on the Graph, and you decide whether to work with that
limitation or prepare an Update.

## When something goes wrong

| Symptom | Fix |
| --- | --- |
| `interf runtime start` prints `Interf local service is already running at http://127.0.0.1:4873, but no bearer token is available.` and `Use the saved connection, pass --token, or restart the service from this CLI.` | Interf Studio's Runtime holds that port. Quit Interf Studio, then run `interf runtime start` again. |
| `interf agents ls` reports `This agent is signed out. Sign in to it, then Interf can use it.` | Sign in to Claude Code, then run `interf agents ls` again. |
| `interf agents ls` reports `Interf cannot find this agent on this Mac. Install it, then refresh.` | Install Claude Code, then run `interf agents ls` again. |
| `interf build` refuses with `Graph fed-watch changed after its Plan occurrence was reviewed.` | The Graph changed after your review, for example after another Scan. Run `interf build-plan history fed-watch` and `interf build-plan show` again, then approve with the new Graph state. |
| `interf graphs files` prints `Choose one of --init, --check or a mapping edit.` | Run `--init`, the mapping edit and `--check` as separate commands. |
| `interf graphs files` refuses the target with `Private folder mappings require an absolute path.` | Write the locator as an absolute path inside double quotes, as in step 6; `~` is refused. |
| `interf graphs files` prints `Graph file changed. Refresh before saving; no changes were overwritten.` | Another writer saved first. Run `interf graphs files "$HOME/fed-watch-graph"` and use the digest it prints. |
| `interf graphs files --init` prints `Private file already exists; read or edit it instead.` | Read it with `interf graphs files "$HOME/fed-watch-graph"`; do not create it twice. |
| `interf graphs files` prints `Private mapping is already tracked by Git. Remove it from shared history before saving.` | The private file must never be shared. Remove `.interf-local.json` from the Git history of that folder. |
| Every item reports `missing` after `--check` | The mapping points at `fomc/` itself. Map the folder that holds `fomc/`, then check again. |
| Every item stays `unchecked` after `--check` | No Source is mapped. Run the mapping step, then check again. |

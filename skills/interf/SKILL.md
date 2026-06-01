---
name: interf
description: "Use when an agent needs to operate Interf from natural language: connect to the local Interf service, manage Projects, review Build Plans, build Context Graphs from source files, and read the returned Context Graph locator for agent tasks."
---

# Interf

Interf prepares data for agent tasks. It lets users' agents build a Context Graph
from their files with source coverage
summaries, task-aware knowledge, graph entrypoints, and links back to sources,
so users can inspect what was covered before agents answer.

Interf is not a search engine and not a general RAG index over every Project
the instance has seen. Treat it as a data-preparation workflow for the user's
current Source and task. Existing Projects and Context Graphs are reusable only
when they clearly bind the same Source and task-relevant Build Plan.

Ask Claude to answer from a folder. With Interf, Claude builds a Context Graph
from the files and the user can inspect what it covered. Interf runs locally,
exposes Graph Manifest metrics, Stage Manifest coverage, traces, and ready /
not ready status, preserves stage execution shells for inspection, and returns
a Context Graph locator agents can read.

## Core Model

- **Instance** = the local service reached through Interf MCP, an installed CLI
  connection, explicit URL/token details, or a developer-started
  `interf runtime`. It owns all Project state.
- **Source** = input files or data. A local folder is the current binding
  shape; Source stays local and read-only, and Interf reads it by reference,
  not by copy.
- **Project** = the durable workspace for one Source binding, agent task
  intent, selected Build Plan, Project activity, execution agents, and the
  latest Context Graph.
- **Build Plan** = the reviewed recipe for how Interf will build a Context
  Graph from the Source for the Project intent. It owns requested outputs,
  expected inputs, coverage metrics, graph entrypoints, Build instructions,
  and optional Source context. It can be drafted, selected, saved, or improved.
  CLI namespace: `interf plan`.
- **Run** = one durable execution record. Build runs, benchmark runs,
  Build Plan draft runs, and Build Plan improvement runs are all Runs.
- **execution shell** = the replayable workspace Interf creates for an
  agent-executed stage. It contains stage-local `AGENTS.md` / `CLAUDE.md`, the
  native skill, contract, expected inputs, Source Manifest, required prior
  outputs, writable output mounts, prompt log, event/status logs, validator,
  and validation attempts. Stage retries create a fresh shell with previous
  attempts and validation feedback mounted under `runtime/`. Use it for
  debugging and retry; do not present it as the product output.
- **Build Context Graph** = the action that runs the selected Build Plan for a
  Project and produces one Context Graph. CLI: `interf build <project-id>`.
- **Context Graph** = source-backed context built from files so agents can use
  prepared context instead of partial file reads. It contains `home.md`, graph
  entrypoints, summaries, knowledge notes, links back to sources, and traces.
  Current local builds expose it as an inspectable folder agents can continue
  from.

Agents should think in Build Plan contracts: "what Context Graph outputs,
expected inputs, coverage metrics, entrypoints, and traces need to exist before
I can answer from this Source without partial file reads?"

## Source Reference Contract

The Context Graph is a knowledge map over the Source. It normally contains
`home.md`, `artifacts/` graph entrypoints, `summaries/` coverage proof,
`knowledge/` navigation/drilldown, traces, and source references. It does not
duplicate the raw Source files.

A built Context Graph has three fixed top-level layers and `home.md`:

- `summaries/` — grounding: one folder per Source file, each linking back to its
  Source. This is COVERAGE proof (every file was read; file-level, not a claim
  that every fact was captured).
- `knowledge/` — the knowledge web: claims, entities, timelines, tables, etc.
  Every note links DOWN to its summary and carries raw `source_refs`, and notes
  link to EACH OTHER. This is TRACEABILITY (every claim links a Source; no
  orphaned summaries; no disconnected islands).
- `artifacts/` — higher-level task outputs that reference summaries + knowledge
  (+ raw source refs for exact claims).
- `home.md` — start here; routes into all three.

Readiness (`ready` / `not ready`) is backed by COVERAGE and TRACEABILITY — both
provable and deterministic. A separate VERIFICATION pass (`interf benchmark`) is
a fallible double-check: it flags claims a verifier could not confirm against the
cited file. It is not proof of absence and does not gate readiness — for exact
claims, always read the original Source via the note's `source_refs`.

Use the Context Graph first to learn what exists, which files/pages/figures are
relevant, which graph outputs were prepared, and where evidence lives in the
original Source. Then follow the graph's source references when exact wording,
table values, chart positions, visual ranges, or provenance matter.

"Answer from the Context Graph" means:

- start from `home.md` as the primary graph entrypoint;
- use generated `artifacts/` as task-specific graph entrypoints;
- use `summaries/` for coverage proof and `knowledge/` for navigation/drilldown;
- use the graph's source references and traces as backlinks to the source of
  truth;
- drill into cited source pages/figures/tables when precision matters;
- keep the answer grounded in the prepared graph plus its cited source evidence.

It does not mean:

- treat summaries or artifacts as a replacement for the underlying Source;
- ignore the original data after the graph points to the relevant evidence;
- make a strong claim from a blurry or under-extracted artifact.

## Non-Negotiable Interf Behavior

If the user invoked `/interf` or explicitly asked to use Interf, the agent must
not bypass Interf by answering from ad hoc raw source-file reads.

Do not tell users to invoke `/anthropic-skills:interf` for the local demo. The
local Interf Skill command is `/interf`. A host-specific managed skill with a
prefixed namespace may be stale and outside this repo's control.

No fallback direct-read answer is allowed. Do not open PDFs, screenshots,
exports, extracted text files, or report bodies to answer the user's question
when Interf is unreachable. Do not say "I could not reach Interf, so I read the
files directly." That is the failure mode Interf exists to prevent.

After Interf has built or selected a Context Graph, opening the graph-cited
source reference for exact verification is allowed and often required. That is
not a fallback. It is the graph's backlink model.

Allowed before a reachable Interf connection:

- check whether Interf is running;
- inspect filenames, extensions, and obvious source-folder shape only if needed
  to identify the Source;
- ask the user to start `interf runtime` / `interf runtime start`, connect the
  agent through Interf MCP, or provide explicit Interf URL/token details.

Required when Interf is unreachable:

```text
I can't answer this through Interf yet because I cannot reach a connected Interf instance.
Please start Interf with `interf runtime` or `interf runtime start`, connect this agent through Interf MCP, or provide the Interf URL/token. Then I will build or inspect the Context Graph before answering.
```

Only answer after one of these is true:

- an Interf Build has produced a Context Graph for the Project;
- a ready existing Context Graph for the same Source and task-relevant Build
  Plan is selected and inspected;
- the user explicitly cancels Interf and asks for a normal direct file read.

## Required User-Visible Flow

When the user asks Interf to read a Source and answer a question, the agent's
first job is to define the data-preparation contract, not to answer. Follow
these gates in order.

Do not satisfy a new question by searching unrelated existing Projects. In the
connected-instance path, create a fresh Project for the current Source and task
unless the user explicitly selected or named an existing Project. If the user
has not provided a Source, ask for the Source instead of answering from whatever
Projects already exist. Project ids are generated internally; do not ask the
user to provide one.

Source scope default: when Interf MCP, the CLI, explicit connection details, or
the user context already provides a Source locator or file set, use that whole
Source. Do not ask the user to choose one file, both reports, or all files
inside that Source unless the user explicitly asks to narrow scope. If no
usable Source locator is available, ask for one Source locator, usually a local
folder path. Do not ask for a file-selection menu.

1. **Propose requested outputs and coverage first.**

   Say what Interf needs to prepare before the answer can be trusted. Requested
   outputs are GRAPH AREAS — claim nodes, comparison nodes, a timeline, a
   coverage index — the next agent will read and reason over. They are NOT
   the answer itself, NOT a finished report, and NOT a summary document. Source
   stays in the user's files; Interf builds the task-specific Context Graph
   the agent traverses. The output is a short in-chat review:

   ```text
   Before I answer, Interf should prepare this Context Graph for the question.

   Context Graph outputs (graph areas, not the answer):
   1. <output name> — <which claim nodes / comparison nodes / timeline Interf will prepare>
   2. <output name> — <same, for the next area>

   Coverage I will verify:
   - files processed
   - source units summarized
   - knowledge reviewed and used
   - graph entrypoints generated

   Approve these outputs and coverage expectations, and I will draft the Build Plan.
   ```

   Stop here unless the user has already explicitly approved this exact set of
   outputs and coverage expectations. Do not answer yet.

2. **Draft the Build Plan after approval.**

   Create or reuse the Project, draft the Build Plan, then summarize it in
   chat. Include the Project id, Build Plan id, requested outputs, stages,
   expected inputs, graph entrypoints, and what the Context Graph will contain.
   Include a review locator when the connected Interf client provides one, but
   the user should be able to review the plan in chat.

   Stop here unless the user approves the Build Plan or already gave explicit
   end-to-end autonomy after reviewing the outputs and coverage expectations.

3. **Build the Context Graph after Build Plan approval.**

   Start the Build, show the Build id and any review locator returned by the
   connected Interf client, then check status by Run id. If the user asked for
   an answer in the same request, continue until the Build reaches a terminal
   state.

   If a Build Plan draft, Build Plan improvement, Build run, or status poll
   reaches `failed` or `cancelled`, stop. Report which Run failed, summarize the
   visible error, and ask the user before starting another draft or Build run.
   Do not automatically retry failed Interf Runs.

4. **Answer only from the Context Graph.**

   Read `home.md` first. Use generated graph entrypoints under `artifacts/`
   as task-specific starting points, `summaries/` as coverage proof, and
   `knowledge/` as drilldown. For exact values,
   quotes, chart reads, tables, or provenance-sensitive claims, follow the
   graph's source references back to the cited Source evidence before answering.
   Answer with the values the Context Graph prepared plus any source-reference
   verification you performed, and say which graph entrypoint and source page
   the answer came from. If the Context Graph points to the right evidence but
   the prepared graph output is too coarse, under-extracted, or missing the required visual
   semantics, say the Context Graph is not ready and improve the Build Plan.

The public package path is CLI/runtime-first. MCP is how agent hosts talk to a
connected Interf instance when the host supports local MCP tools. Use **Interf
MCP** when available, then this Skill as the instruction fallback, then direct
HTTP only with explicit local URL/token details.

Interf MCP lets an outside agent host talk to Interf. Agents listed inside
Interf are executors that run Build Plan stages. Do not treat executor setup as
the same thing as MCP setup.

Use this order:

1. **Interf MCP**, when available.
2. The installed `interf` CLI with an active connection.
3. A developer-started local runtime from `interf runtime` or
   `interf runtime start`.
4. This bundled Skill, when MCP is not available in the agent host.
5. Direct local HTTP only for the well-known discovery probe or when Interf
   explicitly provides the local URL/token.

Do not install packages or use `npx` as part of the normal flow. If MCP is not
available and the CLI has no active connection, ask the user to start
`interf runtime` / `interf runtime start`, run `interf login --url <url>` for a
remote instance, or provide explicit Interf URL/token details.

If MCP is not configured, you may probe `http://127.0.0.1:4873/health` to see
whether the default local runtime is running. Do not scan ports. Do not treat
discovery as authorization. Any mutating or source-reading request needs the
active bearer token from **Interf MCP**, the installed CLI connection, or
connection details explicitly provided by Interf.

Keep MCP and connection probing internal. Do not narrate "CLI is installed",
"discovery returned 200", raw URLs, ports, or token state in the normal demo
flow. User-facing language should be only: Interf connection available, or this
agent is not connected to Interf yet.

When confirming a connection, say that Interf is running. Do not report
package, CLI, or service versions unless the user explicitly asks for debug
details.

## Connection Model

Every mutating action requires a reachable Interf instance.

In the public CLI path, the user starts a local instance with `interf runtime`
or `interf runtime start`. A remote or managed instance can be selected with
`interf login --url <url>` when available. Agent hosts may instead connect
through Interf MCP or explicit URL/token details.

Discovery is not enough by itself. A 200 response from `/health` only means a
local service is running. If discovery succeeds but there is no MCP connection,
no active CLI connection, and no explicit URL/token, stop and ask the user to
connect this agent to Interf.

If the local discovery probe is unreachable, say:

```text
I need Interf running before I can build a Context Graph.
Please start `interf runtime` or `interf runtime start`, then connect this agent through Interf MCP or the installed CLI connection.
```

For local developer sessions with the CLI already installed:

```sh
interf status                    # confirm the active connection
interf runtime                   # fallback: foreground local service
interf runtime start             # fallback: managed background service
```

### Review References

When a Project, Build Plan, or Run is created, include the compact review
summary in chat: Project id, Build Plan id, Run id, requested outputs, primary
metrics, and the Context Graph locator when ready.

The local HTTP URL is an authorized service connection fallback, not a review
UI. Do not present `127.0.0.1` links as the review surface for normal users.

## Agent Flow

1. **Confirm a connection**.

   If Interf MCP tools are available, call `get_instance_status` or
   `instance_status`, depending on the host profile. If only the CLI is
   available, run `interf status`. If neither is available, probe
   `http://127.0.0.1:4873/health` only to check whether the default local
   runtime is running.

   Keep these checks quiet. Do not tell the user which transport worked or
   failed unless they explicitly asked to debug Interf connectivity.

   If Interf provides explicit connection details for this session, pass them
   on each CLI call:

   ```sh
   --url "<service_url>" --token "<auth_token>"
   ```

   Do not write the token into files and do not echo it to the user.

   Do not run hidden runtimes as an automatic recovery step. If MCP cannot
   reach the instance, the CLI has no active connection, and no explicit
   URL/token was provided, ask the user to start `interf runtime` or
   `interf runtime start`, connect through Interf MCP, or provide connection
   details. Stop here. Do not read the Source and do not answer the user's
   question from raw files.

   Treat `package_version`, CLI versions, and raw service metadata as internal
   diagnostics. Do not include them in normal user-facing replies.

2. **Propose what Interf will prepare before mutating state**.

   After confirming an Interf connection, inspect what you can about the
   Source: filenames, file types, obvious page counts, section names, and
   anything that affects the user's question. Treat that as advisory Source
   context, not evidence that Interf processed the files.

   If the user invoked `/interf` or explicitly asked to use Interf, that is
   permission to inspect the Source and propose the graph outputs and coverage
   metrics Interf should prepare. It is not permission to
   answer from the Source. Follow the Required User-Visible Flow above.

   Keep the proposal compact:

   ```text
   I found <source summary>. For <task>, I should build the source first because <risk>.
   <Optional ambiguity or assumption in one sentence.>

   Interf should prepare data for the agent by building a Context Graph from
   the files.

   Context Graph outputs:
   1. **<Plain-English output name>**
      Why it helps: <short sentence>
   2. **<Plain-English output name>**
      Why it helps: <short sentence>

   Coverage metrics:
   - files processed
   - source units summarized
   - knowledge reviewed and used
   - graph entrypoints generated

   Approve these outputs and coverage expectations, and I will draft the Build Plan.
   ```

   Do not show JSON to the user. Do not use markdown tables. Do not include
   extracted numbers, chart readings, schema fields, stage names, raw check
   operator names, or commands in the human proposal. Detailed numbers belong in
   the built artifacts, which live inside the Context Graph.

3. **Create the Project after approval**.

   ```sh
   interf project create <id> --source <path> --intent "<agent task>"
   ```

   Create a fresh Project for the current Source unless the user explicitly
   names an existing Project id. Store the durable agent task on the Project.
   Store requested outputs and advisory source interpretation in the Build Plan
   draft request, using `source_context` when useful.

4. **Draft a Build Plan**.

   For real analytical work, prefer a custom Build Plan for the Project.
   Use the default `interf-default` Build Plan only when the user explicitly
   asks for the default/smoke-test flow or accepts it as a fallback.

   ```sh
   interf plan draft <project-id> \
     --intent "<agent task override, optional>" \
     --requested-artifacts-json '<json-array>' \
     --source-context-json '<json-object>' \
     --artifacts "<optional extra requested output notes or change request>"
   interf runs status <run-id>
   interf plan show <build-plan-id>
   ```

   Do not generate custom shell polling loops. In hosts with app-backed Interf
   MCP tools, use the run-status tool. In CLI-only hosts, run `interf runs
   status <run-id>` as a single command when you need fresh status.

5. **Show the Build Plan review before building**.

   After the Build Plan is drafted, selected, or materially changed, send the
   user a compact in-chat review: Project id, Build Plan id, what Interf will
   prepare, stage names, expected inputs, graph entrypoints, and what each
   stage writes. Include a review locator when the connected Interf client
   provides one, but do not force the user to leave chat to approve. Unless the
   user granted autonomy after reviewing the outputs and coverage expectations,
   pause here.

6. **Build the Context Graph**.

   ```sh
   interf build <project-id> [--watch] [--quiet] [--idempotency-key <key>]
   ```

   Default is async-return: prints the Build id, the Context Graph locator, and
   current `ready` / `not ready` status. With `--quiet`, only the Context Graph locator is printed. With
   `--watch`, the CLI tails events until the Build reaches a terminal state.

   If the user asked for an answer in the same `/interf` request, answer from
   the built Context Graph. Start from `home.md` and graph entrypoints under
   `artifacts/`, then use `summaries/` for Source coverage proof,
   `knowledge/` for drilldown, and traces/source references for exact claims.
   Do not bypass the graph by searching the original PDF/report/source file
   directly.

7. **Inspect Project activity and optionally benchmark**.

   ```sh
   interf runs status <run-id>
   interf runs ls --project <project-id>
   interf runs cancel <run-id>
   interf traces ls --project <project-id>
   interf benchmark <project-id> --target context-graph
   interf benchmark <project-id> --target source-files
   ```

   `interf benchmark` is a benchmark/evaluation run for Q&A / fact-check accuracy.
   Use it when the user wants a source-files baseline, a Context Graph
   evaluation, or a source-vs-Context Graph comparison. It is not the
   readiness surface.

8. **On `not ready`, improve the Build Plan**.

   ```sh
   interf plan improve <project-id> \
     --intent "<what needs to change, optional>" \
     --artifacts "<requested output changes, if any>"
   ```

## Agents

Interf treats the agents it can shell out to, such as Claude Code, Codex, and
custom CLIs, as connected local agents. `GET /v1/instance` reports
`agent_count` and `default_agent`. If `agent_count` is `0`, `interf build` and
Benchmark runs hard-error until the user installs or registers
an agent.

```sh
interf agents ls
interf agents use <name>
interf agents register <name> --command "<cmd>"
interf agents unregister <name>
interf agents map <role> <agent>
interf agents unmap <role>
```

Roles are `extractor`, `summarizer`, `structurer`, `verifier`, and `general`.
Build Plans may declare custom names; the Build runtime treats unknown roles as
`general`. Default behavior is single-active-agent.

## CLI Flags

| Flag | What it controls | When the agent passes it |
|---|---|---|
| `--url <url>` | Override the active connection URL for one call. | Multiple instances or remote target. |
| `--token <token>` | Override the active bearer token for one call. | Explicit connection details, remote instances, or API integration tasks. |
| `--watch` | Block until the Build finishes and tail stages. | Only when the agent must wait inline. |
| `--quiet` | Print only the Context Graph locator on success. | Scripting and agent handoff. |
| `--idempotency-key <key>` | Sent as `X-Interf-Idempotency-Key`. The instance returns the original Build id on retries. | Whenever the agent might retry the same Build. |

## Operating Rules

- Use exact Project ids and Build Plan ids returned by the API. Do not
  invent ids.
- Always confirm a connection exists before running mutating commands.
- If `/interf` was invoked and no connection exists, stop. Do not answer by
  reading raw source files.
- Once a Context Graph exists, use it as the knowledge map. Source drilldown is
  allowed only through graph-provided source references, traces, graph
  entrypoints, or the Project's recorded Source Manifest.
- For long-running work, use the async default and check status by id. Use
  `--watch` only when the agent must wait inline.
- Use MCP when available. Use the CLI only when it is
  already installed. Do not install npm packages in the user's agent session.
- Do not mention npm package versions, CLI versions, or local service versions
  in normal task updates. They are debugging details, not part of the product
  path.
- You may use direct local HTTP for the well-known discovery probe. Do not call
  source-reading or mutating local HTTP APIs unless Interf explicitly
  provides the local connection details or gives an API-integration task.
- Never write helper files into the Source folder. The Source is read-only
  input.
- Treat Build Plan review plus Graph Manifest metrics as the default human
  trust gate.
- Treat structural output checks as necessary but not sufficient for
  precision. If a visual/table/chart answer needs range, axis, period, unit,
  series, or provenance semantics that the graph output did not capture, mark
  the Context Graph not ready and improve the Build Plan.
- The first visible Interf response to a read-and-answer request is the
  requested-outputs-and-coverage proposal. It is not a draft answer.
- Separate proposal from evidence. Before a Build, you can say what Interf
  plans to prepare, which outputs and coverage metrics the Build Plan targets,
  and what graph entrypoints it will produce. After a Build, you can say which
  files/pages were processed, what graph resources were produced, and whether
  the Context Graph is ready.
- Ask one concise clarification only when the Source or agent goal is missing,
  or when ambiguity makes the requested outputs impossible to draft. For
  normal domain ambiguity, state the assumption and make it a check.

## Show Your Work

The Run record, Stage Manifests, Graph Manifest, traces, and Context Graph locator show
what the user can inspect. Whenever you start a Build, summarize the
Project id, Build id, primary metrics, and Context Graph locator in chat. If
the connected Interf client returns a review locator, include it.

Do not present the local HTTP service URL as a review UI.

Example output after `interf build <project-id>`:

```text
Started Build <run-id>.
Project: <project-id>
Context Graph: <locator when available>

I'll check status in a moment with `interf runs status <run-id>`.
```

When a Build finishes, report the final `ready` / `not ready` status, the
Context Graph locator, and the primary metrics or missing coverage that
matter.

## Reading The Context Graph Locator

Build returns a Context Graph locator. With the CLI, `interf build
<project-id>` prints a short multi-line block by default. The Context Graph
locator looks like:

```text
Context Graph: <local-path-returned-by-interf>
```

For `local-path`, use the agent's filesystem tools to read the directory.
Start from `home.md`, then follow graph entrypoints under `artifacts/`. Follow
source refs for exact claims.

## Interf Client Freeform Proposal Mode

When an Interf client asks for a proposal, do not run CLI commands. Write a
single JSON object to the requested output path. Structured controls call the
local service directly; this mode is only for freeform planning.

Use these action types:

- `build`
- `benchmark`
- `benchmark-question-draft` (draft benchmark questions)
- `build-plan-draft`
- `build-plan-improvement`
- `clarification`

Use `build` when the user asks to prepare data, build graph outputs, or refresh
the Context Graph for an agent task. Set `project` to the Project id. Do not set
`build_plan` for a Build proposal; Build always runs the Project's selected
Build Plan. If the request named a different Build Plan, select or draft that
Build Plan before proposing `build`. CLI equivalent:
`interf build <project-id>`.

Use `benchmark` for benchmark/evaluation proposals. They run against the
Context Graph unless the user asks for a source-files baseline. CLI equivalent:
`interf benchmark <project-id>`.

Return this shape:

```json
{
  "action_type": "build",
  "project": "project-id",
  "title": "short approval card title",
  "summary": "one sentence describing which graph outputs will be built",
  "assistant_message": "concise explanation for the user",
  "command_preview": "interf build project-id"
}
```

For Build Plan drafting proposals, keep requested outputs and graph entrypoints
in `values.requested_artifacts[]` for the current API shape, and put the
advisory source summary in `values.source_context`. Use stable output ids.
Treat `values.checks[]` as an optional developer-diagnostic field only when an
Interf client explicitly asks for it; do not make checks the human-facing
proposal.

**Requested outputs are graph areas, not answers.** Each entry must describe a
region of the Context Graph the next agent will traverse — claim nodes,
entity nodes, comparison nodes, timelines, coverage indexes — NOT the user's
answer or a finished report. Interf prepares structured context; the agent
still does the reasoning.

Good titles / purposes (graph-shaped):
- `"London cost-and-take-up claim nodes"` — source-cited claim notes the agent can compare against South West claim nodes.
- `"South West rent-growth signals timeline"` — dated decision/forecast signals extracted from the report.
- `"Page-by-page guide of the report"` — coverage index so the agent knows where evidence for any claim lives.

Bad titles / purposes (replace agent thinking — do NOT use):
- `"NY rent growth answer"` — that is the agent's job, not Interf's.
- `"Final report"` / `"Forecast summary"` — finished answer artifacts.
- `"Recommended decision"` — Interf does not decide; it prepares the data the decider reads.

```json
{
  "action_type": "build-plan-draft",
  "project": "project-id",
  "build_plan": "project-id-build-plan",
  "title": "Draft Build Plan",
  "summary": "Drafts the Build Plan for the requested graph outputs.",
  "values": {
    "build_plan_id": "project-id-build-plan",
    "intent": "London versus South West annual take-up",
    "requested_artifacts": [
      {
        "id": "page-guide",
        "title": "Page-by-page guide of the report",
        "purpose": "Short coverage note per page so the agent knows what evidence lives where."
      },
      {
        "id": "london-claims",
        "title": "London take-up claim nodes",
        "purpose": "Source-cited claim notes (figures, source page, caveat) for London take-up — the agent compares these against the South West nodes."
      },
      {
        "id": "south-west-claims",
        "title": "South West take-up claim nodes",
        "purpose": "Same shape as the London claim nodes, scoped to South West figures."
      }
    ],
    "source_context": {
      "summary": "One 26-page office market report.",
      "observations": ["No obvious section is called South West."]
    }
  }
}
```

Use `clarification` only when the request cannot be mapped safely to one Interf
action.

## Reference

For exact CLI syntax, run the installed command's help:

```sh
interf --help
interf <command> --help
interf <command> <subcommand> --help
```

The installed `interf` command is the source of truth. Do not rely on a copied
command reference when the CLI is available.

Use CLI read commands before asking the user to inspect the UI:

```sh
interf status
interf project show <project-id>
interf plan show <build-plan-id>
interf runs status <run-id>
interf traces show <trace-kind> --project <project-id>
```

The UI link is for visual review. The CLI output is for agent-readable status,
Build Plan review, graph output status, Build errors, Graph Manifest metrics,
Stage Manifest coverage, stage execution sessions, traces, benchmark results,
and ready / not ready summaries.

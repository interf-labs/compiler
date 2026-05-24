# Interf

**Interf prepares data for agents.**

This npm package ships the `interf` CLI and local Interf runtime for building
task-specific Context Graphs from files.

Agents miss things in files. When agents answer from a folder, they first have
to discover what is in it, decide what matters, extract evidence, and connect
facts across files. That file-to-context step is hidden, inconsistent, and hard
to inspect.

Interf separates that step. It makes users' agents build a Context Graph from
their files. It has source coverage summaries, task-aware knowledge, artifact
handoffs, and links back to sources. Your agent uses the prepared map instead
of rediscovering the files during the task.

```text
Source files                        Context Graph agents use

bristol-office-market/              <context-graph>/
  q4-market-report.pdf                AGENTS.md
  lease-comps.xlsx                    home.md
  planning-notes.md                   summaries/
  exports/availability.csv            knowledge/
                                      artifacts/
                                      traces/
```

## What a Build Produces

A Build produces a Context Graph. The Context Graph is source-backed context
built from files so agents can use prepared context instead of partial file
reads.

For a local Build, the Context Graph is an inspectable folder:

```text
<context-graph>/
  AGENTS.md              # agent guidance
  home.md                # overview and routes
  artifacts/             # task-specific agent handoffs
  summaries/             # source coverage proof
  knowledge/             # navigation and drilldown
  traces/                # links back to sources
```

The output is not an answer. It is a prepared map over the Source: artifact
handoffs, summaries, knowledge notes, and links back to sources. The Source
remains the ground truth. Agents start from `artifacts/`, use `summaries/` for
coverage proof and `knowledge/` for drilldown, then follow source links when
exact wording, table values, chart reads, or provenance-sensitive claims matter.

## Design Choices

- `Project-scoped`: every Project starts from a specific Source and selected
  Build Plan, not a generic index over every file.
- `Inspectable`: Interf records what was built, which requested Artifacts exist,
  and where source-backed traces live.
- `Local-first`: local Builds keep Source files on your machine and read-only.
- `Bring your own agent`: use Claude Code, Codex, or another registered
  command-line agent.
- `File over hidden index`: local Builds expose the Context Graph as a folder
  agents can inspect.
- `Context Checks you control`: every Build can be checked against
  plain-English conditions such as "Every page is covered" or "Every figure
  cites a source page."

## Why Not Just Ask Your Agent?

You can. Interf can use Claude Code, Codex, or another registered local agent
while it builds the Context Graph.

A one-off preprocessing prompt gives you another answer to trust. Interf puts
the preparation step inside a Build Plan you can inspect: requested Artifacts,
declared stages, Build evidence, Context Checks, and traces.

The agent can still execute the stages. Interf shows what was covered, what was
produced, and whether the Context Graph is ready for the task.

## Install

```bash
npm install -g @interf/compiler
interf            # opens the wizard
```

Requires Node.js 20+ and a local agent CLI such as Claude Code, Codex, or
another registered command-line agent. Run `interf doctor --live` if the
executor is not detected.

## Quick Start

```bash
# terminal 1: start the local Interf runtime
interf runtime

# terminal 2: create a Project from a local Source
interf project create bristol --source ./bristol-office-market

# draft the Build Plan for review
interf plan draft bristol \
  --intent "Bristol annual take-up and availability chart lookup" \
  --artifacts "Guide to the report; annual take-up figures with source references" \
  --ready-when "Every page is listed and every figure has a source reference."

# select the reviewed Build Plan
interf plan select bristol <build-plan-id>

# build the Context Graph
interf build bristol

# optional: benchmark/evaluate answers from the Source baseline, Context Graph, or both
interf benchmark bristol
```

`interf runtime` starts the local runtime in the foreground and writes the
active connection record so subsequent CLI commands can connect. Agents can use
`interf runtime start` for an explicit managed background runtime, then
`interf runtime stop` when their task is done.

Mutating commands never implicitly auto-start a runtime. If no instance is
connected, they exit with a hint pointing at `interf runtime`,
`interf runtime start`, or `interf login`.

`interf build` returns the Context Graph locator on success. For local Builds,
that locator points to a folder agents can inspect and continue from. There is
no `--out` flag and no implicit copy of your Source folder.

## Context Graph

The Context Graph is the output Interf builds from your files for agents. It is
a knowledge map over the Source, not a replacement for the Source.

It gives agents structure and source-backed routes for navigating files. For
the built-in `interf-default`, it includes:

```text
<context-graph>/
  AGENTS.md       # agent-facing guidance and source-checking rules
  CLAUDE.md       # same guidance for Claude Code
  home.md         # graph index
  artifacts/      # task-specific handoffs for agents
  summaries/      # one source-named folder per source file
  knowledge/      # linked notes built from summaries and source refs
  traces/         # source provenance for claims and checks
```

The source files stay the source of truth. Interf writes generated state inside
the instance data directory and does not modify the Source.

`AGENTS.md` tells agents how to use the Context Graph: start from the prepared
outputs, follow the prepared routes, and use the recorded source references
when exact source evidence matters.

## Build Plans

A Build Plan tells Interf how to build requested Artifacts from your Source for
the agent task. It also states the Context Checks the user can review before the
Build.

The built-in `interf-default` Build Plan ships with `interf`. Save your own
local Build Plan with `interf plan save <path>`; draft new ones with
`interf plan draft <project-id>`.

```text
<build-plan-folder>/
  build-plan.json         # Build Plan definition: artifacts, stages, build rules
  build-plan.schema.json  # output contract: required Context Graph files/folders
  README.md               # what this Build Plan is for
  build/
    stages/
      summarize/          # stage instructions Interf runs during the Build
      structure/
      shape/
  use/
    query/                # how agents read the Context Graph
  improve/                # how Interf revises the plan if checks still fail
```

- `build-plan.json` is the technical filename for the Build Plan definition:
  requested Artifacts, stages, and how the files should be built.
- `build-plan.schema.json` describes the output contract: what the Context Graph
  must contain.
- `build/stages/<stage>/` is where you author one folder per stage.
- `use/query/` holds the instructions agents follow when they read the Context
  Graph for the task.
- `improve/` holds the instructions Interf follows when the first Build misses
  and the Build Plan itself needs editing.

## Build Plan Improvement

When the first Build is `not ready`, Interf can edit the Build Plan and build
again. Same Source, same checks, improved Build Plan and Context Graph.

```bash
interf plan improve <project-id>
```

Interf records Build Plan improvement as a Run with the revised Build Plan,
Build evidence, and resulting Context Graph.

## Context Checks And Benchmarks

Context Checks are plain-English promises the user reviews before a Build.
Requested Artifacts back those checks.

Examples:

- every file in scope was processed
- required pages or slides were inventoried
- required outputs exist
- every figure cites a source page
- the Source has not changed since the Build

Benchmarks are different. A benchmark asks saved questions and measures answer
accuracy against an agent source-access baseline, the Context Graph, or both.

```bash
interf benchmark bristol
interf benchmark bristol --target source-files
interf benchmark bristol --target context-graph
```

`interf benchmark` is optional evaluation. It is not the main readiness surface.

## What Interf Is Not

- Not a second brain or memory product. One Project starts from one Source and
  agent task.
- Not a vector store or hosted RAG server. The Context Graph is structured
  source-backed context agents can inspect.
- Not a hosted data platform by default. Local Builds run on your machine.
- Not a generic agent task orchestrator. Interf prepares files into Context
  Graphs for agents.

## Useful Commands

- `interf` / `interf init` - open the interactive wizard.
- `interf runtime` - run the local Interf runtime in the foreground.
- `interf runtime start` - start a managed background local runtime.
- `interf runtime stop` - stop the running local runtime.
- `interf project ls / create / show / rm` - manage Projects.
- `interf plan list / show / save / draft / select / improve` - manage Build
  Plans.
- `interf build <project-id>` - Build the Context Graph for a Project.
- `interf graphs ls --project <id>` / `show <graph-id> --project <id>` - inspect
  Context Graphs.
- `interf traces ls --project <id>` / `show <trace-kind> --project <id>` -
  inspect Context Graph traces.
- `interf runs ls --project <id>` / `status <run-id>` - inspect Runs.
- `interf benchmark <project-id>` - run an optional benchmark/evaluation pass.
- `interf agents ls / use / register / unregister / map / unmap` - configure
  local agents and role mapping.
- `interf login / logout` - connect the CLI to another Interf instance.
- `interf status` - show the active connection and Project summary.
- `interf doctor --live` - check the local executor before a Build.

## Public Assets

- [skills/interf](./skills/interf/) - bundled agent Skill for Interf.
- [build-plans/interf-default](./build-plans/interf-default/) - default Build
  Plan that ships with `interf`.

Contributors: see [CONTRIBUTING.md](./CONTRIBUTING.md).

License: see [LICENSE.md](./LICENSE.md). © 2026 Interf Inc. All rights reserved.
Name and branding: see [TRADEMARKS.md](./TRADEMARKS.md).

# Interf

**Interf prepares data for agent tasks.**

Interf structures your files into a task-specific graph your agent can navigate.
Your agent starts from prepared context built for the task, instead of a partial
read of whatever files it happened to open.

> This npm package is the `interf` CLI and local runtime. A Mac app (Interf
> Desktop) is coming soon.

## Quick start

```bash
npm install -g @interf/compiler
interf                 # the interactive wizard walks you through your first build
```

Requires Node 20+ and a local agent CLI (Claude Code, Codex, or another). Check
it with `interf doctor --live`.

Or run it command by command:

```bash
interf runtime                                 # start the local runtime
interf project create bristol \                # bind a folder + the agent's task
  --source ./bristol-office-market \
  --intent "Bristol annual take-up and availability"
interf plan draft bristol                      # draft a Build Plan, then review it
interf plan select bristol <build-plan-id>     # pick the reviewed plan
interf build bristol                           # build the graph
```

`interf build` returns the graph path: a folder your agent opens and continues
from. No `--out` flag, no copy of your Source. Mutating commands never auto-start
the runtime; they exit with a hint if nothing is connected.

## What you get

A folder your agent navigates instead of grepping raw files. Interf calls it a
**Context Graph**:

```text
<graph>/
  home.md          # start here. routes into the three layers below
  summaries/       # one folder per source file. every file read and summarized
  knowledge/       # the connected web: linked notes, claims, entities, source refs
  artifacts/       # task handoffs that reference summaries and knowledge
  traces/          # provenance: every claim links back to source
  AGENTS.md        # how agents use the graph (plus CLAUDE.md for Claude Code)
```

`summaries/`, `knowledge/`, and `artifacts/` are the three fixed layers, plus
`home.md`. What goes inside `knowledge/` (claims, entities, timelines, tables) is
up to the Build Plan, so any task shapes its own web. Your Source stays read-only
and the ground truth. Agents follow source links when exact wording, table
values, or chart reads matter.

## Why Interf is different

Most tools prepare your files one way for every question: chunks in a vector
store, or a single knowledge graph that maps everything the same way (RAG
indexes, Obsidian-style graphs, code-graph tools). Interf is `task-specific`. You
tell it what the agent needs to do, and it builds a graph tailored to that task:
the right summaries, connections, and entrypoints for *this* work, not a one-size
index of all your files.

And the graph is `provable`:

- `Coverage`: every Source file was read and summarized.
- `Traceability`: every claim links back to a Source, and every note connects to
  the web. No orphans, no islands.

It is `provider-agnostic` by design: the same prepared context works across
Claude, Codex, GPT, and whatever you switch to next, because the graph is a
portable folder you own, never trapped in one vendor's session. Your own agents
do the work, your files never leave your machine, and the output is files you can
read, diff, commit, and reuse.

## Design principles

- `Not a replacement for your agent harness`: just a task-specific graph to navigate.
- `Your files stay yours`: local, read-only, never sent to Interf's servers.
- `Runs with your agents`: your CLIs, your subscriptions, in a replayable shell.
- `File over app`: the graph is inspectable files, not a hidden index.
- `Connected, not orphaned`: every note links to others and back to sources.
- `Coverage-first`: exact file, summary, knowledge, and output counts.
- `Project-scoped`: one Source, one agent task, one Build Plan.

## Readiness is provable

Every Build reports exact coverage metrics (files processed, source units
summarized, knowledge reviewed and used, graph outputs, entrypoints) behind a
single `ready` / `not ready` verdict backed by two deterministic guarantees:

- **Coverage**: every Source file was read and summarized. This is file-level. It
  does not claim every fact was caught, only that no file went unread.
- **Traceability**: every claim links back to a Source, every summary is linked,
  and every knowledge note connects to the web. No orphans, no islands.

If a file is unread, a claim is unlinked, or a note is an island, the graph is
`not ready` and the gap is named.

`interf benchmark` is a separate, optional check that scores answer accuracy
against the Source baseline, the graph, or both. It is a fallible double-check,
never a readiness gate.

## Build Plans

A Build Plan is the reviewed recipe for how Interf builds the graph from your
Source for the task: requested outputs, stage instructions, expected inputs, and
the entrypoints you review before building. `interf-default` ships built in.
Draft your own with `interf plan draft <project-id>`, or save one with
`interf plan save <path>`.

When a Build comes back `not ready`, Interf can revise the plan and build again
(same Source, same intent, better plan), recorded as a Run:

```bash
interf plan improve <project-id>
```

A Build Plan is a folder you can read and version:

```text
<build-plan>/
  build-plan.json         # outputs, stages, and build rules
  build-plan.schema.json  # the output contract: what the graph must contain
  build/stages/<stage>/   # the instructions Interf runs for each stage
  use/query/              # how agents read the graph for the task
  improve/                # how Interf revises the plan when checks still fail
```

## Commands

| Command | What it does |
|---|---|
| `interf` / `interf init` | Open the interactive wizard |
| `interf runtime` / `start` / `stop` | Run, background, or stop the local runtime |
| `interf project create / ls / show / rm` | Manage Projects |
| `interf plan draft / select / improve / save / list / show` | Manage Build Plans |
| `interf build <id>` | Build the graph for a Project |
| `interf graphs ls / show --project <id>` | Inspect built graphs |
| `interf traces ls / show --project <id>` | Inspect source provenance |
| `interf runs ls / status --project <id>` | Inspect Runs |
| `interf benchmark <id>` | Optional accuracy check |
| `interf agents ls / use / register / map` | Configure local execution agents |
| `interf status` | Connection and Project summary |
| `interf doctor --live` | Check your agent before a Build |

## What Interf is not

- Not a second brain or memory product: one Project, one Source, one task.
- Not a vector store or hosted RAG server: the graph is inspectable, source-backed files.
- Not a hosted data platform: local Builds run on your machine.
- Not a general knowledge graph or notes app: each graph is built for one agent task, not a single index of everything.

## More

- [skills/interf](./skills/interf/): the bundled agent Skill.
- [build-plans/interf-default](./build-plans/interf-default/): the default Build Plan.
- [CONTRIBUTING.md](./CONTRIBUTING.md) · [LICENSE.md](./LICENSE.md) · [TRADEMARKS.md](./TRADEMARKS.md)

© 2026 Interf Inc. All rights reserved.

# Interf

Give your agents the context they need.

Category: Reusable context preparation for agents.

This repository holds the open parts of Interf: the Context Interface
specification, its schemas and an example, and the Interf skill for agents. The
compiler itself, `@interf/compiler` on npm, is free to use with your own agents.

## The problem

An agent starts each session without the context the work needs. It searches and
re-reads raw files, often reads only part of them, and can miss a file or a
detail that matters. When it answers, you cannot see what it used or what it
missed. The next session starts over.

## The solution

Declare what the work needs, prepare it once, and reuse it.

1. **Declare.** A Context Interface is one JSON file that says what context the
   work needs: the kinds of knowledge, how they relate, the outputs and where an
   agent should start. It works like `package.json` for context: it names what
   you need, not how to get it.
2. **Prepare.** The compiler has your agent read your Sources, read-only, and
   builds a Graph that satisfies the Interface: a folder of Markdown files with
   source coverage, source links and traces. You review the Interface, then the
   exact Build Plan, before anything is built.
3. **Reuse.** Your agents start from the Graph instead of rediscovering raw
   files. When Sources change, an Update prepares a new version of the Graph.

Coverage means every selected file is accounted for. It does not prove that every
fact was understood.

## The contract

- [Specification](context-interfaces/SPEC.md): the Context Interface document,
  the Graph package and Context protocol version 1 (release `context-protocol-v1`).
- JSON Schemas: [Context Interface](context-interfaces/context-interface.schema.json),
  [Source requirements](context-interfaces/graph-requirements.schema.json) and
  [private Source mapping](context-interfaces/graph-local.schema.json).
- [Example](context-interfaces/examples/component-maintenance/README.md): a
  Context Interface for component maintenance work.
- [Interf skill](skills/interf/SKILL.md): teaches your agent the reviewed
  preparation loop.

A Context Interface is strict JSON (`"kind": "graph-shape"`, version 3), and
unknown fields fail. It holds no Source paths, prompts, credentials or executable
instructions. Its identity is a sha256 digest of its canonical JSON. A Graph is
either source-linked (agents also need the original Sources) or self-contained
(the Graph carries the knowledge it needs).

A finished Graph travels as one ZIP with five files: `README.md`, `AGENTS.md`,
`.gitignore`, `graph.tar.gz` (the exact accepted Graph) and
`source-requirements.json`, its dependency file. Original Source files are never
included, and any agent can use the package without installing Interf. The
dependency file pins every cited Source item version. Each recipient checks its
own copies, and each item reports `exact`, `changed`, `missing`, `unavailable` or
`unchecked`. These are reported checks, not proof of access or freshness.

## Install and first use

Needs an Apple silicon Mac and Node.js 22.19 or later. This page describes
`@interf/compiler` 0.51, which ships with the October 2026 release; the 0.50 on
npm today is an earlier generation with different commands. Check
`interf --version`.

```sh
npm install --global @interf/compiler
interf runtime start    # the local Runtime, on your device
interf help             # every command, grouped: set up, organize, prepare, observe
```

Then connect the agent you already use, in one of three ways:

- **Skill:** give your agent the [Interf skill](skills/interf/SKILL.md). It
  runs the whole loop through the CLI.
- **MCP:** in your agent host's stdio MCP config, run `interf` with the arguments
  `["mcp", "--profile", "app"]`.
- **CLI:** `interf agents ls`, then `interf agents connect`; the skill shows the
  request file.

Now ask your agent to prepare a Graph for a piece of work from a folder. It scans
the folder, drafts the Context Interface and the Build Plan, and stops twice for
you: approve the exact Interface, then the exact Build Plan with its estimated
cost. Only then does it build. The result is a Graph your agents can read, and a
ZIP you can hand to any agent.

Local work with your own agent needs no Interf account. Interf Studio, the Mac
app, wraps the same compiler and is optional.

Without installing anything, you can check a Context Interface with any JSON
Schema tool, compute its digest with the Node recipe in the specification, and
give a delivered Graph ZIP to any agent.

## What is open and what is not

- **Open, Apache-2.0:** the specification, schemas and example in
  `context-interfaces/`, and the Interf skill in `skills/`.
- **Proprietary:** everything else, including the compiler, Interf Studio and
  the [Runtime API description](openapi/README.md). The compiler ships as a
  compiled package and is free with your own agents. See [LICENSE.md](LICENSE.md)
  and [TRADEMARKS.md](TRADEMARKS.md).
- **Local by default:** the Runtime runs on your device. Sources stay where they
  are, read-only; connecting one creates no copy or upload. Telemetry is
  counts-only and on by default; `interf telemetry opt-out` turns it off and
  deletes the events still queued on your device.
- **Optional and opt-in:** Interf Agent, Interf's own preparation agent, is
  metered and uses credits. The cloud environment prepares Graphs off your device
  only when you choose it, from Sources you explicitly upload; no remote worker
  reads local files across the network. [Interf Intelligence](intelligence-api.md)
  gives an optional recommendation from the reviewed task and category labels
  you send.

## Contributing and security

See [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md).

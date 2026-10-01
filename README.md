# Interf

Give your agents the context they need.

Category: Reusable context preparation for agents.

## The loop

```text
state the work
→ add read-only Sources
→ prepare, review, and approve the Context Interface
→ prepare a Build Plan
→ review and approve the exact Plan
→ build or update the Graph
→ use the accepted Revision with your agent
```

The Runtime records coverage, Source links, traces, readiness, and immutable
Graph Revision lineage.

## Surfaces

- **Interf Studio**: the macOS visual Client and local Runtime host
- **CLI**: an agent-friendly Client for local or remote Runtime profiles
- **MCP**: connects an external agent host to the selected Runtime
- **Interf skill**: teaches your agent the reviewed preparation loop
- **Interf Platform**: managed Graph Build metered by credits

All surfaces use the same Runtime API. None duplicates Graph semantics.

- [Interf Intelligence API](intelligence-api.md): an explicit recommendation step before private Context Interface preparation.

## Install the published CLI

This installs the latest version published on npm, which may lag the source tree:

```sh
npm install --global @interf/compiler
interf --help
```

`@interf/compiler` is the package name; `interf` is the command and Interf is
the product.

Give your existing Agent the [Interf instructions](skills/interf/SKILL.md) to
connect, prepare and use a Graph. Studio is optional; check the installed
version's help for its supported commands.

## Runtime contract

The Context Interface and Build Plan are separate review stops, in that order.
The public Context Interface contract is in
[`context-interfaces/`](context-interfaces/README.md).

## Privacy and execution

Local Source stays local and read-only by default. The engine consumes
agent-built Source inventories and never reaches around the Source access
adapter.

Remote managed execution requires an authorized read-only Source
materialization beside the Worker. Workers propose; the selected Runtime
validates and commits. No remote Worker silently reads local files across the
network.

## Contributing and security

See [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`SECURITY.md`](SECURITY.md).

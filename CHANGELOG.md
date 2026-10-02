# Changelog

> For agents: use only the commands, flags and fields on this page, exactly as written. Run `interf --version` first; if it prints a different version, use `interf --help` and `interf <command> --help` instead of this page. llms.txt at the repository root lists every page.
>
> Checked against `@interf/compiler` 0.51.0.

Each Context protocol release is an annotated Git tag on this repository, and a
tag never moves. A release can add documentation, examples and schema metadata.
It never changes the wire versions, so files delivered under an earlier release
stay valid and keep citing it.

## context-protocol-v1.1 (2026-10-02)

Additive. The wire versions are unchanged: `graph-shape` 3,
`interf-graph-requirements` 1 and `interf-graph-source-mapping` 1 and 2. Graph
folders delivered under `context-protocol-v1` stay valid.

- Each JSON Schema declares its `$id` at this tag.
- The [specification](context-interfaces/SPEC.md) has a Graph folder section:
  the folder layout, the five dependency statuses and the exact protocol lines
  every new `AGENTS.md` carries.
- New `AGENTS.md` files cite this release and name `npx @interf/compiler` for use
  without an installation.
- New example: [Fed watch](context-interfaces/examples/fed-watch/README.md), for
  FOMC statements and minutes. Each example README states its digest.
- New tutorial: [Prepare your first Graph](first-graph/README.md).
- [llms.txt](llms.txt) lists every page. Each page starts with an instruction
  block for agents and the compiler version it was checked against.
- The [Interf skill](skills/interf/SKILL.md) shows `npx @interf/compiler` and
  states that Interf Agent uses credits while your own agent is free.
- The README states the problem, the contract, install and what is open. The
  security policy, the Intelligence API page and the OpenAPI document were
  updated.

## context-protocol-v1 (2026-09-30)

First publication: the Context Interface specification and its JSON Schema, the
Source requirements and private Source mapping schemas, the component
maintenance example and the Interf skill.

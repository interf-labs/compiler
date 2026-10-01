# Contributing

This public surface contains the docs and agent artifacts users need to inspect.
Do not add private maintainer paths, local machine paths, company plans, or
website source here.

## Public projections

- Product and install projection: [`README.md`](README.md)
- Security: [`SECURITY.md`](SECURITY.md)
- Agent workflow: [`skills/interf/SKILL.md`](skills/interf/SKILL.md)
- Strict output ABI: [`context-interfaces/`](context-interfaces/README.md)

## Contract rules

- Graph is the top-level product object.
- Sources remain read-only and reusable.
- A Context Interface document (wire noun `graph_shape`) is source-path-free,
  task-instance-free, strict, and explicitly versioned.
- A Graph-scoped approval selects the exact reviewed Interface digest.
- A Build Plan pins each selected Source's exact Inventory and references that
  Interface approval.
- Execution requires separate approval of the exact Plan.
- Graph Revisions are immutable; folders are replaceable materializations.
- Public docs never point to maintainer-only or private repository paths.

## How changes land

This directory is a published projection, not a buildable project: it ships no
`package.json`, no test runner, and no scripts, so there is nothing to run here
before opening a PR.

Propose wording or contract changes as an issue or a PR against these files.
The maintainer release gate is what validates them, and it runs two checks that
matter for anything you send:

- the public doc layout check, which keeps this directory the single source for
  the packaged docs rather than a copy of them
- the OpenAPI generation check, which re-derives the document under
  [`openapi/`](openapi/) from the Runtime's own operation table

[`openapi/`](openapi/) is therefore generated. Edit the Runtime contract, not
the JSON: a hand edit is overwritten by the next generation, never adopted.

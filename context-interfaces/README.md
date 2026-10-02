# Context Interface

> For agents: use only the commands, flags and fields on this page, exactly as written. Run `interf --version` first; if it prints a different version, use `interf --help` and `interf <command> --help` instead of this page. llms.txt at the repository root lists every page.
>
> Checked against `@interf/compiler` 0.51.0.

This Apache-2.0 public protocol and the [Interf skill](../skills/interf/SKILL.md)
also describe delivered Graph folders; see [Context protocol version 1](SPEC.md#context-protocol-version-1).
The current release is `context-protocol-v1.1`; the [changelog](../CHANGELOG.md)
lists every release.

A Context Interface is the exact strict blueprint that defines the logical
output one Graph Build must produce. The wire format and schema keep the
`graph_shape` noun; everything else carries the Context Interface name.

It is:

- source-path-free
- validated with unknown fields rejected
- embedded exactly inside a reviewed Build Plan
- descriptive, not executable

The standard flow is task + Sources → prepare and approve the Interface →
prepare and approve the Build Plan → build the Graph. The approved Plan digest
binds the selected Interface, task, pinned Source Inventories, and ordered
stages.

See [`SPEC.md`](SPEC.md) for the document contract. Examples:
[Fed watch](examples/fed-watch/README.md) and
[component maintenance](examples/component-maintenance/README.md).

## Publish and reuse

An application can distribute a `context-interface.json` as a dependency
manifest describing the Graph it needs. A person can prepare a Graph against
that same file. Share the file with a description-only README; it contains no
Source selection, prompts, credentials, or executable instructions.

Validate a saved file without a Runtime connection, account, Graph, or Sources:

```sh
interf context-interface validate --file context-interface.json
interf context-interface validate --file context-interface.json --json
```

Without an installation, run the same command as
`npx @interf/compiler context-interface validate --file context-interface.json`.

Success prints the canonical document digest; invalid documents fail. Validation
checks the Interface document, not whether a Graph satisfies it or is Ready.
Loading the file into a Graph creates a candidate for review and approval:

```sh
interf context-interface load <graph-id> --file context-interface.json
```

[`context-interface.schema.json`](context-interface.schema.json) is generated
from Interf's current typed document schema. It provides structural checks for
editors and JSON Schema tools, including required fields, field types, id
patterns, and unknown-field rejection. It cannot express all Runtime validation:
safe output paths, unique ids, valid references, output overlap, and the required
entrypoint are checked by the CLI validator above. Use the CLI for full validation.

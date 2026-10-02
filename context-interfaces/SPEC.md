# Context Interface: document specification

> For agents: use only the commands, flags and fields on this page, exactly as written. Run `interf --version` first; if it prints a different version, use `interf --help` and `interf <command> --help` instead of this page. llms.txt at the repository root lists every page.
>
> Checked against `@interf/compiler` 0.51.0.

Status: public contract summary. Public name: **Context Interface**. The wire
format and code keep the frozen `graph_shape` noun. Releases are listed in the
[changelog](../CHANGELOG.md).

## Document

One Context Interface is one strict canonical JSON document:

```json
{
  "kind": "graph-shape",
  "version": 3,
  "consumption": "source-linked",
  "kinds": [
    { "id": "claim", "label": "Claim", "description": "A grounded assertion." },
    { "id": "evidence", "label": "Evidence", "description": "A Source-grounded fact." }
  ],
  "relationships": [{ "id": "supported-by", "label": "Supported by", "from": "claim", "to": "evidence" }],
  "outputs": [{ "id": "home", "path": "notes", "kind": "directory", "contains": ["claim", "evidence"], "required": true }],
  "entrypoint": "home"
}
```

Unknown fields fail validation. Kind, relationship, and output ids are unique.
Relationship endpoints and output `contains` values name declared kinds. Output
paths are unique, non-overlapping, safe Graph-relative paths. `entrypoint` names
one required output.

Optional `consumption` declares `source-linked` (using the Graph requires
authorized original Sources and private recipient mappings) or `self-contained`
(the task's knowledge is represented in the Graph and use requires no original
Sources). Both retain Source provenance and both are built from Sources. The
absence of this field means source-linked without adding any bytes or changing
historical digests. Explicit declarations participate in canonical identity.
Generated Interfaces declare the admitted choice; loaded Interfaces stay exact.
This is the bounded additive admission to version 3; older strict readers refuse
the new field. Neither declaration alone proves semantic sufficiency.

The document has no self-declared id, revision, label, digest, counts,
cardinality, filesystem-format rules, validation policy, Source selection,
executable instructions, credentials, or model/provider fields. Canonical JSON bytes and
their digest are copy identity. A Graph-scoped approval record binds the exact
reviewed digest as selection identity. Version 2 is not accepted.

## Graph index

A candidate Graph carries one strict logical `graph-index` document. Nodes name
an Interface `kind_id`, an Interface `output_id`, one output-owned path, and
exact `{source_id,file_id}` references. Relationship instances name one declared
Interface relationship and existing endpoint nodes. One node inside the
Interface entrypoint output is the actual entrypoint.

Node paths are unique. A file output owns only its exact path and permits at
most one node; a directory output owns descendant paths and permits distinct
nodes there. Use a directory or multiple file outputs for linked node instances.
An output's `contains` lists allowed kinds, not required counts or a requirement
to instantiate every listed kind. Declared relationships allow typed links but
do not require instances. Each required output must contain at least one node,
and every node must be connected to the actual entrypoint through declared
relationships, ignoring direction only for this connectivity check.

Runtime validation checks the index against the approved Interface and the
exact per-Source Inventory pins in the approved Build Plan. Counts are observed
by grouping nodes by arbitrary `kind_id`; no fixed kind-count ABI exists.
Filesystem frontmatter, filenames, Markdown links, and renderer heuristics are
not Interface satisfaction authority.

## Boundary and distribution

An Interface defines what the finished Graph must contain. It is
source-independent and contains no Source ids, paths, task values, Build stages,
mutable external pointers, scripts, prompts, or credentials.

A distributable Interface directory has one authoritative
`context-interface.json`, a README, and optional non-authoritative examples. Git
may provide provenance; only canonical bytes and Runtime approval provide
identity and selection authority.

## Build Plan and Revision binding

A reviewed Build Plan embeds the exact approved document under `graph_shape`.
The Plan pins each Source Inventory individually, assigns every selected Source
file to exactly one ordered stage, and records an optional base Revision id. It
cannot replace the Interface at Plan-authoring time.

An accepted Graph Revision records the approved Plan identity. The exact
Interface is therefore reachable through the immutable Plan. Changing the
Interface requires a new Plan approval and produces a new Revision when the
candidate passes Runtime validation and commits.

Folders are replaceable materializations of an accepted Revision. They are
never Graph identity or Runtime authority.

## Check a receiving application's requirement

Compute the digest of the application's independently required document with
`graphShapeDigest` from `@interf/compiler/contracts/graph-shape`, or
`interf context-interface validate --file context-interface.json --json`.
Pass it as `expected_interface_digest` to
`RuntimeClient.inspectGraphRevisionDependencies(graphId, revisionId, options)`
or MCP `revision_dependencies`; CLI uses `graphs dependencies <revision-id>
--graph <graph-id> --expected-interface-digest <digest> --json`.

The existing dependency read returns the pinned `consumption` and
`context_interface.document`, its canonical `digest`, and `check`:
`not-checked` without a requirement, or `match`/`mismatch`
with `expected_digest`. The actual document comes from that exact accepted
Revision's verified pinned Plan, including brought Graphs without producer Runs.
Unavailable or unverified evidence refuses the read; it is not a mismatch.
Mismatch means choose a matching Revision or prepare one with the required
Interface. Names, labels, entrypoints and structural similarity do not establish
exact identity. A match proves neither semantic fitness nor Source access.

The portable companion carries the same document and digest. For file-only use,
check the selected Revision and manifest digest against the package you intended
to receive, validate the document and recompute its canonical digest before
comparing with your own requirement. This assumes you trust the accepted package;
self-consistent downloaded bytes alone do not establish authenticity. Private
Source mappings do not supply Interface evidence or alter the requirement.

### Canonical digest without an Interf installation

Use this same recipe for the independently required document and the companion's
`context_interface.document`:

1. Parse JSON and validate the strict `graph-shape` version 3 document above.
   Validation rejects invalid or unknown fields; it adds no defaults, coerces no
   values, trims no text and performs no Unicode normalization. Preserve string
   contents and array order exactly.
2. Recursively sort every object's keys in ascending UTF-16 code-unit order
   (JavaScript's string `<` comparison, not locale order). Recurse into array
   elements without reordering them. The schema's object keys are nonnumeric;
   native integer-index key enumeration does not affect this document.
3. Serialize the sorted value with compact ECMAScript `JSON.stringify` semantics:
   no indentation, BOM or trailing newline. Use its native string escaping,
   including escapes for quotes, backslashes, control characters and lone
   surrogates; do not ASCII-escape other Unicode characters.
4. Encode that JSON text as UTF-8, hash those bytes with SHA-256, and prefix the
   64 lowercase hexadecimal digits with `sha256:`.

For example, after strict document validation, this standalone Node.js code
computes the identity without importing Interf:

```js
import { createHash } from "node:crypto";

function sorted(value) {
  if (Array.isArray(value)) return value.map(sorted);
  if (value === null || typeof value !== "object") return value;
  return Object.fromEntries(Object.keys(value).sort().map(key => [key, sorted(value[key])]));
}

const digest = "sha256:" + createHash("sha256")
  .update(JSON.stringify(sorted(document)), "utf8").digest("hex");
```

First require the recomputed companion document digest to equal
`context_interface.digest`; only then compare it with the independently required
digest. Bind the selected Revision and manifest digest to the trusted accepted
package as described above. A valid hash alone does not establish that trust.

## Context protocol version 1

Current release: `context-protocol-v1.1`. Each release is an immutable Git tag
on this repository; the [changelog](../CHANGELOG.md) lists them. A release never
changes the wire versions (`graph-shape` 3, `interf-graph-requirements` 1,
`interf-graph-source-mapping` 1 and 2), so a Graph folder delivered under
`context-protocol-v1` stays valid and keeps citing that release. The protocol
specification and [Interf skill](../skills/interf/SKILL.md) are Apache-2.0; see
[LICENSE.md](LICENSE.md).

### Graph folder

```text
graph-folder/
  README.md                 how to use this Graph
  AGENTS.md                 the same guidance for agents, with the protocol lines below
  .gitignore                keeps the private file, its lock and temporary saves out of Git
  graph.tar.gz              the exact accepted Graph
  source-requirements.json  interf-graph-requirements version 1
  output/                   accepted files extracted from graph.tar.gz
  .interf-local.json        private mapping and reported checks, version 2
  .interf-local.json.lock   present only while a writer saves
```

The shareable ZIP holds the first five files. Extract accepted files into a
separate `output/` child. Local Graph Open materializes that child beside the
same companions. Delivery guidance does not modify accepted output, its archive
digest, or its Context Interface. Only the recipient creates `.interf-local.json`,
and only on request.

Each cited item version reports one status:

| Status | Reported when |
| --- | --- |
| `unchecked` | No matching reported check: not checked yet, nothing mapped, or the target or requirements changed after the check |
| `exact` | A reported read whose hash equals the cited version's hash |
| `changed` | A reported read whose hash differs |
| `missing` | The mapped location has no file at the cited path |
| `unavailable` | The mapped location could not be read |

Each status is a reported observation with its recorded time, not proof of
current access, permission or freshness. A self-contained Graph needs no checks.

Every `AGENTS.md` written under this release carries these protocol lines
verbatim, after the requirements' instructions. Folders delivered under
`context-protocol-v1` carry the same lines without the `npx` line.

```text
Keep accepted output unchanged. The delivery directory is not accepted Graph content.
The only recipient state is .interf-local.json: kind interf-graph-source-mapping, version 2, revision, requirements_digest, entries and checks.
Copy revision from these requirements, using manifest_ref.digest as manifest_digest. Compute requirements_digest using the canonical JSON/SHA-256 recipe in the requirements instructions.
Entries use the version 1 mapping fields described below; expected_local_source is null for a new standalone entry. Start checks as an empty array.
A reported check identifies source_id, inventory_id, inventory_digest, file_id, requirements_digest, target_digest and observed_at. Compute target_digest from the exact entry target with the same canonical recipe.
Its result is {outcome: read, content_hash: sha256:...}, {outcome: missing} or {outcome: unavailable}. Use valid JSON with quoted names and values.
A matching reported hash means exact bytes at the recorded time; a different hash means changed. No matching check means unchecked. A changed target or requirements invalidates earlier checks.
These editable reports are not authenticated current access, freshness, permission or accepted evidence. Explain gaps; the user may choose limited use. Self-contained use needs no original checks.
Create this private file with owner-only permissions. Keep it ignored and untracked. Refresh before editing. To save, exclusively create .interf-local.json.lock, reread the requirements and exact prior file digest, refuse a stale read, then atomically replace the private file. Remove only the lock inode you created.
Agents using their own file tools must follow that same lock protocol or serialize edits with Studio/CLI. An editor that ignores the lock can race the final replacement; do not edit simultaneously in that editor.
Optional CLI: interf graphs files <delivery-directory> --json. Add --init to create the private file; --source <id> --target '<JSON target or null>' --expected-digest <digest or absent> to edit; --check to explicitly check mapped cited local files.
Without an Interf installation, run the same commands as npx @interf/compiler graphs files <delivery-directory>; never npx interf, which is a retired package.
No Runtime, account, Source registration, Scan or Update is required for these file operations. An Agent can use its own authorized tools instead.
```

The optional CLI reads, creates, maps and checks, with no Runtime. One command
cannot combine `--init` and `--check`. A `local-folder` locator must be an
absolute path; `~` is refused.

```sh
interf graphs files /delivered/graph --json
interf graphs files /delivered/graph --init
interf graphs files /delivered/graph --source <source-id> --target '{"kind":"local-folder","locator":"/authorized/folder"}' --expected-digest <prior-digest-or-absent>
interf graphs files /delivered/graph --check
```

### Requirements and the private file

`source-requirements.json` is `interf-graph-requirements` version 1.
Its [structural schema](graph-requirements.schema.json) pins the accepted Revision,
Context Interface document/digest, consumption mode, entrypoint and every cited
Source item version with output references. No recipient paths, local registrations,
credentials or reported checks belong in that shared file.

For `self-contained` consumption, read the represented Graph knowledge without
finding or checking originals. For `source-linked` consumption, use only authorized
locations to check the cited versions. Report limitations before work; a user may
proceed with explained limitations. Neither choice creates access permission,
accepted evidence, freshness or factual correctness.

The optional recipient-owned `.interf-local.json` lives in the delivery directory,
outside `output/`. It is `interf-graph-source-mapping` version 2; see the
[structural schema](graph-local.schema.json). Keep it owner-readable/writable only,
ignored and untracked by Git, and excluded from every export. Create it only at
the recipient's explicit request. Version 1 mapping files remain valid for explicit
Runtime import. Importing does not register, scan or update Sources.

Each local entry maps one logical `source_id` to a credential-free `target`:
`{"kind":"local-folder","locator":"/authorized/folder"}`, an authorized
`agent-location` locator, or `null`. Preserve `expected_local_source` from an
exported mapping. Remove a prior `local_source` selection when changing its target.
An absent registration is not missing file evidence.

Each reported check binds `requirements_digest`, `source_id`, `inventory_id`,
`inventory_digest`, `file_id`, `target_digest`, and `observed_at`.
The result is `{"outcome":"read","content_hash":"sha256:…"}`,
`{"outcome":"missing"}`, or `{"outcome":"unavailable"}`.
Use the canonical digest recipe above for both complete validated
requirements and the exact target object. Distinct retained versions remain distinct
checks even if their logical Source and file IDs match.

Refuse a private file whose Revision does not match exactly. For a valid file,
the deterministic projection is `unchecked` unless the file/check requirements
digests and target digest match.
With matching identities, a read hash equal to the required item hash is `exact`;
a different hash is `changed`. A missing/unavailable result retains that status.
Always label these as reported observations with their recorded time. Hand-edited
checks are not authenticated proof or present access, permission, freshness,
accepted readiness, or semantic sufficiency.

JSON Schema covers structure only. Full validation additionally verifies the
Context Interface canonical digest and its supplied comparison, agreement between
Interface consumption and instructions, unique logical mapping entries/check keys,
the exact private-file Revision, and known logical Sources. To independently
compare an application requirement, preserve the supplied requirement and compute
its own digest; without one, report `not-checked`. A saved supplied comparison
never substitutes for this receiving application's comparison.

The CLI and connected Studio read the same file and use the same pure projection.
Studio offers explicit refresh, mapping save and mapped-file checks. Protocol
writers exclusively create `.interf-local.json.lock`, reread the requirements and
exact prior file digest, refuse stale reads, then atomically replace the private
file. Remove only the lock inode that writer created, including after failure.
Agent file tools must follow this same protocol or serialize their edits with
Studio/CLI. A lock-unaware editor can race the final replacement; it must not edit
simultaneously. This is cooperative locking, not filesystem compare-and-swap. Runtime
mapping import is a separate explicit action. Offline file reads reuse the 64 MiB
Agent workspace file bound; the native bridge retains its 1 MiB transport bound.
Use CLI or authorized file tools for metadata too large for that bridge.

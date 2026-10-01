---
name: interf
description: "Prepare, review, build, update, and inspect Interf Graphs through the selected Runtime. Use this when the user wants the agent to work from a source-backed Graph instead of rediscovering files every time, including registering or refreshing a Source, refreshing a Source inventory, approving a Context Interface or a Build Plan, starting a Graph Build or an Update, following a Run, or reading a Revision's readiness, coverage, source links, traces, and entrypoints. Triggers on: Interf, Interf Studio, Graph, Context Interface, graph_shape, Build Plan, Graph Revision, Source, Source inventory, coverage, source links, readiness, traces, \"prepare context for my agent\"."
license: Apache-2.0
---

# Interf

Use Interf when the user wants an agent to work from a source-backed Graph
rather than repeatedly rediscovering files.

## Safety

- Treat Sources as read-only.
- Use only the Runtime connection the user or Interf Studio authorized.
- Never scan ports or invent a service URL.
- Scan, Interface and Plan preparation are Agent jobs: run them only within the
  user's authorized preparation and provider/spend limits. Build or Update
  execution additionally requires review and approval of the exact Build Plan.
- Never treat a folder or Worker result as committed Graph truth.
- For source-linked consumption, never claim a Graph is ready for an Agent's
  work until that receiving Agent has checked its own current access to every
  Source item cited by the selected Revision. The user may explicitly proceed
  with explained access limitations;
  that choice does not verify historical evidence or grant new permissions.

## Workflow

If Interf is not installed, install the user-approved package version or artifact
with npm; the published package is `@interf/compiler`:
```sh
npm install --global @interf/compiler
interf --help
```
Published releases may lag the source checkout. Use the intended installed
version's help and tool schemas, not commands assumed from another version.

Select the authorized Runtime explicitly. Use the running Studio Runtime, or
start a local Runtime with `interf runtime start` when authorized. Keep an existing
authorized connection and its private credentials. `interf connect` replaces the
saved connection, including its token; do not use it merely to check access.
When given an explicit URL, add `--url <authorized-url>` to each CLI command below.
CLI-only setup needs no MCP-host configuration or reload. If using MCP instead,
configure the agent host's existing stdio MCP integration to run `interf` with
arguments `["mcp", "--profile", "app"]`, adding `"--url", "<authorized-url>"`
when supplied. Use its existing private connection credentials; never paste a
Runtime bearer into a prompt. The MCP server does not implicitly start a Runtime.

Interf account sign-in and vendor CLI authentication are separate. Local work
with the user's own configured Agent does not require a new Interf account
sign-in. When the requested account or Cloud operation needs it, use
`interf auth login --json` and follow `interf auth status --json`, or MCP
`auth_login_start` and `auth_login_progress`. Complete only the intended account
verification with the user's authorization; report pending or refused access.
This does not authenticate the vendor CLI or authorize paid Interf execution.

Run `interf agents ls --json` first (MCP: `agents_list`) and reuse an appropriate
authorized Agent. To connect an installed, configured Codex, write a connection
request file with the intended paths:
```json
{"type":"codex","auth":{"mode":"cli","customer_context":{"project_root":"/your/project","home":"/your/home"}}}
```
Run `interf agents connect --request agent.json --json`, or call MCP
`agent_connect` with the same object. Check the returned connection and readiness;
after an uncertain response, refresh the roster before trying again.
Set `config_home` and `profile` inside `customer_context` when your vendor setup
uses them. These are paths and names, never credentials. Bare `cli` connections
remain Controller-only.
This explicit mode starts a fresh installed CLI session with your skills, MCP,
vendor configuration and permissions; it requires no Docker or second sign-in.
It is customer-controlled execution and provides no Interf OS isolation.
If your tools require shell environment variables, explicitly start the Runtime
from that configured environment with `interf runtime start`; an already-running
Desktop Runtime cannot inherit another terminal's private variables. Never copy
vendor credentials or silently broaden permissions or project trust. Interf
still validates outputs and alone accepts Graph revisions; vendor authentication
and configured tools must actually succeed before claiming execution works.

Connecting does not select job defaults. Inspect the registry for the existing
choices and readiness. Interface preparation, Plan preparation and Plan review
use the `context-interface-preparation`, `build-plan-preparation` and
`graph-build` job defaults respectively. Reuse suitable existing choices. Only
when the user authorizes remembering a choice, write that choice to a JSON file:
```json
{"agent_id":"<connected-agent-id>"}
```
Run `interf agents default context-interface-preparation --request choice.json
--json` once per needed job kind. MCP `agent_set_job_default` takes the same
choice plus `job_kind`.
This replaces that job's saved choice: preserve any intended supported `model`,
`effort` and `fast_mode` fields in the same object. Do not silently overwrite or
clear existing defaults. Scan can use an explicit Agent id without changing its
`source-scan` default. Verify readiness before submitting each job.

1. Confirm the selected Runtime is reachable and authorized.
2. State the user's work in their words.
3. Select an existing Graph for that work, or create one before adding Sources:
   ```sh
   interf graphs ls
   interf graphs show <graph-id>
   interf graphs create <graph-id> --intent "<exact work>"
   ```
   For inspection or use of an existing Graph, continue at steps 12–13. Source
   refresh, preparation, Build and Update are separate actions; reading an
   accepted Revision does not require them.
4. Register or select each relevant Source, then add it to the Graph. Record
   the Source id printed by `sources add`:
   ```sh
   interf sources ls
   interf sources add local-folder <path>
   interf graphs source-add <graph-id> <source-id>
   ```
5. Ensure each Source has an accepted Inventory. If its last observation is
   seven or more elapsed days old, offer a refresh or obtain explicit approval
   to use that exact Inventory; a known mismatch must be refreshed.
   ```sh
   interf sources show <source-id>
   interf sources scan <source-id>
   ```
   `sources scan` only reads the accepted Inventory; it does not execute a Scan.
   When the first Scan or a refresh is authorized, run
   `interf sources refresh <source-id> --agent-id <agent-id>` (MCP:
   `source_inventory_refresh` with `source_id` and `agent_id`). Poll its returned
   Run with `interf runs status <run-id>` until terminal, then inspect the accepted
   Inventory. A failed or merely submitted Run is not accepted evidence.
6. Ask Interf to prepare a Context Interface from the work, follow its Run, and
   review the exact candidate before approval:
   ```sh
   interf intelligence recommend <graph-id> --dry-run --json
   interf intelligence recommend <graph-id> --json > recommendation.json
   interf context-interface generate <graph-id> --recommendation recommendation.json
   interf runs status <run-id>
   interf context-interface ls <graph-id>
   interf context-interface review <graph-id> <graph-input-digest> <interface-digest>
   ```
   To reuse a saved Interface, use `interf context-interface load <graph-id> --file context-interface.json`, then review its candidate before approval.
   Choose consumption explicitly when generating: CLI `--consumption source-linked`
   or `--consumption self-contained`; MCP `context_interface_generate` accepts the
   same `consumption` values. Omission selects source-linked. Self-contained
   prepares represented knowledge that consumers use without original Sources;
   source-linked requires authorized originals. Both build from Sources and retain
   provenance. Loading an existing Interface preserves its declaration and bytes.
   Recommendations are optional. Preview the exact work and accepted category
   labels, then explicitly send them with `intelligence_recommend` or the command
   above. Interf and its model provider receive only that request; Interf retains
   it until account deletion. User-authored text may identify someone. Never add
   filenames, paths, excerpts, Instructions or a private draft. Supply the strict
   receipt to `context_interface_generate` as `recommendation`, or use null/omit
   the CLI file option to prepare privately without recommendations. The authoring
   Agent receives reference material and makes no Interf recommendation call.
7. Present its exact kinds, relationships, outputs, and entrypoint; obtain
   explicit approval for the exact Interface digest. Only after that approval,
   record it with:
   ```sh
   interf context-interface approve <graph-id> <graph-input-digest> <interface-digest>
   ```
8. Prepare a Build Plan from the approved Interface and exact pinned Source
   Inventories. With no accepted Revision or approved Plan, use
   `interf build-plan create <graph-id>`. With an accepted Revision, use
   `interf update <graph-id>`, including when only the approved Interface changed.
   If a Plan is already approved before the first Build, revise it with
   `interf build-plan improve` using its exact saved version from Plan history
   and the required options in `--help`. Follow the returned Run, then review
   and approve the new Plan before execution:
   ```sh
   interf runs status <run-id>
   interf build-plan history <graph-id>
   interf build-plan show <graph-id> <build-plan-id> <plan-digest>
   ```
9. Present the complete Plan, including its ordered stages, exact file
   assignments, executing agent, and resource/spend boundary.
10. Obtain explicit approval for the exact Plan digest. Only after that
    approval, use the Agent, state version, pricing receipt, and any Inventory
    acknowledgements printed during review:
    ```sh
    interf build-plan approve <graph-id> <build-plan-id> <plan-digest> --expected-state-version <version> --agent-id <agent-id> --reviewed-pricing <receipt>
    ```
    Human CLI approval output prints the exact `interf build` command. For CLI
    JSON, use `interf build --help`; for MCP, use the `graph_build` tool schema.
    Supply the returned approval identifiers and Graph state version, keeping
    the Agent and pricing receipt already reviewed. Do not repeat approval just
    to obtain human output.
11. Follow the Run with `interf runs status <run-id>` (MCP: `get_run_status` with
    `run_id`) until terminal, then check the accepted Revision's readiness.
    Report failure or cancellation without automatically retrying.
12. Resolve the current accepted Revision and read its Source requirements:
    ```sh
    interf graphs show <graph-id> --json
    interf graphs show latest --graph <graph-id> --json
    interf graphs dependencies <revision-id> --graph <graph-id> --json
    ```
    MCP exposes the same read as `revision_dependencies`. It returns the exact
    pinned Context Interface under `context_interface`. If the receiving
    application requires an Interface, compute its independently supplied
    document's digest with `interf context-interface validate --file <file>
    --json`, then pass `--expected-interface-digest <digest>` on the dependencies
    command (MCP: `expected_interface_digest`). Check `context_interface.check`:
    `match` is exact identity, `mismatch` calls for a matching Revision or normal
    shared-Interface preparation, and no requirement reports `not-checked`.
    Unavailable/unverified evidence refuses; never replace the application's
    requirement with the Graph's own digest. This check is separate from Source
    access, freshness and semantic fitness. The same read returns the exact
    entrypoint, logical Source/item references, relative paths, accepted content
    hashes and linked outputs. This is the one central lookup for the distinct
    observed `(source_id, inventory_id, digest, file_id)` versions cited by this
    Revision, not every scanned or reviewed-but-unused item. Retained outputs may
    cite an older version alongside the current one: keep both requirements,
    using their exact Inventory pins and `output_refs` to distinguish them.
    It works for brought Graphs without the producer's
    execution records. You may inspect accepted Graph output while Source access
    is unresolved. `graphs inspect` / `revision_inspect` provides the fuller
    inspection when that Runtime retains the execution records; never invent
    missing traces.
    `graphs open <graph-id>` opens the OS file browser when native inspection is
    available; opening it does not prove that this agent read the files.

    Follow the selected Revision's `consumption` and returned instructions.
    For `self-contained`, work from represented Graph knowledge without locating,
    mapping or reading originals; the complete cited-version list retains build
    provenance. Report knowledge gaps instead of assuming unavailable facts.
    For `source-linked`, use safe filename and relative-path hints only within authorized
    locations, then check candidate bytes against the expected hashes. Report
    exact, changed, missing and unchecked items separately and explain their
    effect on the requested work. The user may explicitly proceed with those
    limitations; never label changed or unchecked material as verified historical
    evidence. This requires no Scan or Update and grants no additional access.
    Accepted coverage and a Graph read confirmation do not prove Source access.

    To fetch a portable Graph into a user-visible location, choose an explicit
    new file in an authorized existing destination folder:
    ```sh
    interf graphs archive <revision-id> --graph <graph-id> --portable --output /chosen/folder/graph.zip --json
    ```
    The destination must not already exist; relative paths resolve from the CLI's
    working directory. This is an explicit export location, not a named workspace
    or a promise about Runtime cache paths. MCP `revision_archive_read` returns
    base64 archive bytes, not a file at a requested path; use the CLI export for
    this portable ZIP, or the host's authorized file tools for MCP-returned bytes.
    The ZIP contains `README.md`, `AGENTS.md`, `.gitignore`, the exact accepted `graph.tar.gz`, and portable
    `source-requirements.json` without local mappings or producer locators.
    Extract into a new authorized folder and follow that README. Unpack the inner
    archive separately; read the requirements' entrypoint relative to its output
    folder. Keep the requirements outside the accepted output folder. Send the
    whole ZIP. Neither an archive alone nor its requirements prove
    current Source access or freshness; exporting does not confirm another
    agent's read. A file-only Agent follows the JSON's same consumption
    instructions, including reporting limitations and the user's explicit choice
    to proceed. Its recipient needs no Interf
    installation, account or Runtime Source registration: keep a private local
    mapping for source-linked consumption inside the delivery directory but outside accepted output and shared exports, and use the Agent's own
    authorized tools. A new location alone does not prove that its material
    matches the accepted evidence. An exact hash proves matching bytes, not
    safety, semantic correctness or freshness.

    Follow [Context protocol version 1](../../context-interfaces/SPEC.md#context-protocol-version-1)
    and its strict requirements/private-file schemas. The matching release
    is `context-protocol-v1`.
    The recipient-created `.interf-local.json` is version 2, owner-only, ignored
    and untracked. Agents, the CLI and connected Studio edit that same file.
    Each reported check binds the complete requirements digest, logical Source,
    exact Inventory/file version, target digest and observation time. Derive
    `unchecked`, `exact`, `changed`, `missing` and `unavailable` from those
    identities; changed targets or requirements invalidate prior checks.
    A reported hash match proves neither present access nor freshness, and
    hand-edited checks are not authenticated accepted evidence.

    ```sh
    interf graphs files /delivered/graph --json
    interf graphs files /delivered/graph --init
    interf graphs files /delivered/graph --source <source-id> --target '{"kind":"local-folder","locator":"/authorized/folder"}' --expected-digest <prior-digest-or-absent>
    interf graphs files /delivered/graph --check
    ```

    Use an explicit refresh before saving external edits; stale saves refuse.
    File-tool writers exclusively create `.interf-local.json.lock`, reread the
    requirements and prior file digest, refuse stale reads, then atomically replace
    the private file and remove only their own lock inode. Serialize lock-unaware
    editor changes with Studio/CLI; they can race the final replacement.
    Checking reads only cited files through authorized mappings and never
    registers or copies Sources. A self-contained Graph needs no such check.
    Version 1 mapping import remains supported. Runtime reconciliation is a
    separate explicit import, preserving the exported registration baseline.

    For source-linked consumption, resolve each original Source once through the environment's central local
    mapping to an authorized folder or remote connection. Reuse that mapping
    for all its cited items. In a Runtime, inspect missing mappings and ask the
    user which existing Source here corresponds to each original Source reference.
    Map or clear it explicitly:
    ```sh
    interf sources ls
    interf graphs source-map <graph-id> <revision-id> <original-source-id> --to <local-source-id>
    interf graphs source-map <graph-id> <revision-id> <original-source-id> --clear
    ```
    MCP exposes `graph_source_map`. Resolve each stable `(source_id, file_id)`
    reference in Graph notes through the shared requirements and local mapping;
    retain its exact observed version. One mapping does not make both historical
    and current bytes available. Qualify Runtime cited reads with `node_id` when
    multiple versions exist; ambiguous unqualified reads and mismatched bytes
    refuse, even when the user chooses to work with changed material.
    Relative paths are readable hints. A mapping preserves accepted Markdown
    and original Source/item identities; never rewrite notes with recipient paths.
    It neither registers a Source nor changes Sources selected for the next Build.
    Register a new connection only with the user's authorization, then map it.
    A replacement Source incarnation needs a new mapping. Matching names or
    relative paths does not prove matching content.

    With explicit authorization, use `interf sources check-access <local-source-id>
    --agent-id <agent-id>` or MCP `source_access_check` to ask the selected Agent
    about its own access. This starts an advisory Run, not a read-only inspection
    or accepted Scan; managed execution uses credits. Poll that Run before
    reporting its result. A successful Source-level advisory result alone does
    not prove access to every cited item; it does not replace the complete check
    above. When updating from replacement local Sources, explicitly select them
    and obtain fresh accepted Scan evidence before preparing the new Plan.
    An Interface-only Update can reuse unchanged accepted Inventories, subject
    to the freshness checks in step5; unchanged pins do not prove current file
    freshness.
13. Report only the readiness, coverage, Source links, traces, and entrypoints
    returned by exact-revision Runtime reads. Report that accepted Revision
    readiness separately from semantic fitness and, for source-linked consumption,
    the receiving Agent's current item access checks.
    Report any explicit decision to work with limitations without claiming full
    verified access. Do not substitute a placement-local latest folder for
    immutable revision truth.

## Recovery

- Interface rejected: revise or select another candidate; do not begin Plan
  preparation without an approval.
- Plan rejected: prepare a revised Plan; do not weaken the strict contract.
- Source Inventory mismatch: refresh that Source, prepare a new Plan, and
  review again. Age alone is advisory and may be acknowledged for the exact
  accepted Inventory.
- Source moved or unauthorized during source-linked consumption: report the unresolved items,
  resolve an authorized connection through the central local mapping, then have
  the receiving Agent repeat its access check for every cited item. Preserve the
  accepted Graph; never search outside the granted location.
- Self-contained consumption continues from represented Graph knowledge without
  original Sources or mappings. Unavailable originals limit optional verification,
  not use of that knowledge; report any gaps in the Graph itself.
- For preparation or Update with a replacement Source, register it only with
  the user's authorization, explicitly select it and obtain a fresh accepted
  Scan before reviewing a new Plan.
- Agent unavailable: connect an authorized agent, then rerun the admission
  check.
- Build failed: preserve the accepted prior Revision and completed evidence;
  inspect the failure, then ask the user to recover it in Desktop or prepare a
  replacement Plan through the currently exposed actions.

## Domain

- Graph is the top-level product object.
- Source inventory belongs to its Source and can be reused across Graphs.
- Context Interface is a strict output contract with its own explicit approval.
- Build Plan is execution authority.
- Graph Revision is immutable accepted state.
- A folder is only a materialization of one revision.

Use `interf --help` or the connected MCP tool descriptions to discover the
exact supported commands/resources for the installed version.

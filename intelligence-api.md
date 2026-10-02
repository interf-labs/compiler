# Interf Intelligence API

> For agents: use only the commands, flags and fields on this page, exactly as written. Run `interf --version` first; if it prints a different version, use `interf --help` and `interf <command> --help` instead of this page. llms.txt at the repository root lists every page.
>
> Checked against `@interf/compiler` 0.51.0.

Get optional recommendations before privately preparing a Context Interface.

```http
POST https://interf-cloud.vercel.app/api/intelligence/v1/recommend-interface
Authorization: Bearer <Interf session token or itf_ak_* key holding graphs:write>
Content-Type: application/json

{ "request": { "version": 2, "task": "<the work>", "categories": ["<accepted label>"] } }
```

The strict request has only version, task (at most 4 KiB) and up to 128 category
labels, ordered without duplicates. Its canonical JSON is limited to 64 KiB.
Draft candidates, Instructions, filenames, Source identifiers and other fields
are rejected.

The response is `{ "outcome": "hit", "graph_shape": { ... }, "result": { ... } }`
or `{ "outcome": "no-match" }`. The result is reference material, never approval.
A `request_id` may accompany `request`; omitting it derives one from the account
and exact request bytes. Exact replay returns the retained result without another
allowance debit. A different request counts separately. Team identities are not
served by this version.

## Scan, recommend, prepare

The Graph Runtime offers a preview and explicit send at
`/api/runtime/v1/graphs/{graph_id}/interfaces/recommendation`.
GET returns `{ request, request_key }`, grounding labels in accepted inventories.
POST takes only `{ request_key }`, rechecks those exact outgoing bytes and returns
one strict receipt value containing the request, existing request id/key, origin
provenance and hit/no-match result. Changed input pins with identical grounded
outgoing bytes do not invalidate a preview.

Use `interf intelligence recommend <graph-id> --dry-run --json` to inspect;
`interf intelligence recommend <graph-id> --json` sends. The controller MCP tools
are `intelligence_preview` and `intelligence_recommend`.
Save the receipt JSON and pass `--recommendation <file>` to `context-interface
generate`, or pass the value as `recommendation` to `context_interface_generate`.
Null/omitting the CLI option prepares without a recommendation. Generation freezes
that reference in its existing request archive. It sends no Intelligence request,
draft or automatic recheck. Origin records where the reference came from; it can
remain relevant after task or Source changes.

## Disclosure and usage

Only the reviewed task and category labels go to Interf Intelligence and its model
provider, under the provider's terms. Interf retains each request and recommendation
until account deletion. Interf adds no filenames, paths, Source ids, excerpts,
raw data, Instructions or private Interface. Text chosen by the caller may itself
identify a person or company.

GET on `recommend-interface` with `request_id` reads retained usage for the current
account. Caller-supplied recommendation data is not proof of provider spend.
Unavailable usage remains unknown; reusing a recommendation does not multiply its
physical provider calls.

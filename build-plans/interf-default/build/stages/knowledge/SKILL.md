# Knowledge

Build the task-aware Context Graph knowledge layer from the summaries.

Contract type: `build-knowledge`

## Requirements

- Treat the Context Graph knowledge layer as retrieval support, not final truth.
- Read `runtime/project.json` and `runtime/stage-contract.json.project.intent`
  before choosing the knowledge notes. The knowledge layer must be task-aware:
  extract claims, entities, timelines, comparisons, risks, or indexes that help
  the saved Project intent, not just a generic source outline.
- Read `runtime/expected-inputs.json` before writing notes and write
  `runtime/reviewed-inputs.json` with one decision for every required expected
  summary before DONE.
- `runtime/reviewed-inputs.json` must use this shape:
- `{`
- `  "kind": "interf-stage-reviewed-inputs",`
- `  "version": 1,`
- `  "generated_at": "2026-01-01T00:00:00.000Z",`
- `  "stage_id": "knowledge",`
- `  "reviewed": [`
- `    { "resource_id": "summary-example", "decision": "used", "source_refs": ["summaries/example/summary.md"] }`
- `  ]`
- `}`
- Prefer durable entity, claim, and index notes over one giant catch-all file.
- Write those notes as flat markdown files under `knowledge/` unless this
  Build Plan explicitly declares typed child folders.
- The Context Graph has three FIXED top-level layers plus `home.md`, each with a
  purpose serving humans AND agents:
  - `summaries/` — grounding/coverage: one folder per Source file, each linking
    back to its Source. Human-first: navigate and understand "what's in my data".
  - `knowledge/` — the knowledge web: claims, entities, timelines, tables, etc.
    Agent-first: traverse and query. Every note links DOWN to its summary, carries
    raw `source_refs`, and links to OTHER knowledge notes.
  - `artifacts/` — higher-level task outputs/handoffs that reference summaries and
    knowledge (plus raw source refs for exact claims). Human deliverable and agent
    entrypoint.
  - `home.md` — start here; routes into all three.
- The interior of `knowledge/` is free: a typed interior such as
  `knowledge/entities/`, `knowledge/claims/`, `knowledge/timelines/`,
  `knowledge/tables/`, or `knowledge/atlas/` is encouraged when the task needs
  richer structure. But core derived content must stay UNDER these canonical
  layers — never invent a new top-level folder (for example `graph/` or
  `insights/`) for core content. If you need richer structure, nest it under
  `knowledge/<interior>/`; the layer recognition routes `knowledge/<anything>/`
  to the knowledge layer.
- Keep claims connected to the summaries and original source references that
  support them, so users can follow links back to sources.
- Every knowledge note MUST wikilink the summary of every source it cites.
  For each source a note records in its `source_refs`, add a
  `[[summaries/<source-name>/summary]]` wikilink (or
  `[[summaries/<source-name>/manifest]]` for paginated/visual sources) in that
  note's body. Source refs alone are not enough: a note that cites a source
  without linking its summary leaves that summary an orphaned island and fails
  the stage. This is not optional — there is no plain-source-ref opt-out for the
  summary layer.
- Build the knowledge layer as a connected WEB, not isolated islands. In
  addition to the mandatory `[[summaries/...]]` down-link, every knowledge note
  should link at least one OTHER knowledge note (entity to claim, claim to
  related claim, note to index) using its filename stem under `knowledge/`. A
  note that links no other knowledge note is a disconnected island and fails the
  `knowledge_web_connectivity` check. This enforces CONNECTEDNESS, not a count: a
  single genuine link is enough, and a genuinely lone note on a sparse Source is
  fine. Never invent links to pass — fabricated links are worse than honest
  sparsity.
- Every claim or entity note MUST also record raw `source_refs` in JSON
  frontmatter — the original Source paths it asserts from — so a reader can drop
  to ground truth and verify instead of trusting a paraphrased summary. Keep
  both pointers: the `[[summary]]` wikilink is for navigation, the `source_refs`
  are for verification, and every source in `source_refs` must also have its
  summary wikilinked (above).
- A pure index or navigation note that asserts nothing of its own (for example a
  `knowledge/index.md` route map, or the no-intent `knowledge/task-focus.md`
  fallback) may set `"note_role": "index"` in its JSON frontmatter to opt out of
  the `source_refs` requirement. Do not use this to dodge provenance on a note
  that makes a claim — fabricated source refs are worse than none.
- Use knowledge notes as navigation and assurance. They are not the authority
  for exact wording, table values, chart reads, or provenance-sensitive claims.
- Keep knowledge-stage links stage-local: prefer linking only to summaries or knowledge notes that are already meaningful by the end of this stage. Avoid relying on `home.md` or other entrypoint-stage routes as the main navigation surface.
- Prefer direct file-reading and search tools over shell commands for routine file inspection.

## Notes

- Bias knowledge notes toward canonical entities, claims, timelines, and stable indexes.
- Use taxonomy and ontology only as means to improve retrieval, navigation, and evidence tracking.
- For small Sources, prefer a minimal stable substrate over exhaustive flow sprawl.
- When you add wikilinks, target real Context Graph notes by exact basename or explicit relative path. Do not invent title-style links unless that exact title is also a declared note label or alias.
- Do not add wikilinks in knowledge outputs to files that the entrypoint stage creates later. If a route belongs in `home.md` or a later entrypoint note, leave plain text now and let the later stage add the link.
- Build task-aware claims, entities, topics, timelines, comparisons, and indexes from the summaries and their source refs. Use clear filenames such as `knowledge/rent-growth-claims.md`, `knowledge/market-entities.md`, or `knowledge/index.md`.
- If the Project intent is missing, write a small `knowledge/task-focus.md`
  note that says the graph has no saved Project intent yet and bias the rest of
  the knowledge layer toward coverage and source routing.
- For summary references, you MUST link the summary itself: `[[summaries/<source-name>/summary]]` or, for paginated/visual sources, `[[summaries/<source-name>/manifest]]` (page note links are an optional addition). Do not cite a summary's source with plain source refs only — link the summary note. For links to other knowledge notes, use the final filename stem under `knowledge/` — and use these to satisfy the web requirement above: every knowledge note should reach at least one other knowledge note so the layer is a connected web, not a star of islands flagged by `knowledge_web_connectivity`.

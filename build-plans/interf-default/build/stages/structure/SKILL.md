# Structure

Build the task-aware Context Graph structure from the summaries.

Contract type: `build-knowledge-structure`

## Requirements

- Treat the Context Graph structure as retrieval support, not final truth.
- Prefer durable entity, claim, and index notes over one giant catch-all file.
- Write those notes as flat markdown files under `knowledge/` unless this
  Build Plan explicitly declares typed child folders.
- Keep claims connected to the summaries and original source references that
  support them, so users can follow links back to sources.
- Use knowledge notes as navigation and assurance. They are not the authority
  for exact wording, table values, chart reads, or provenance-sensitive claims.
- Keep structure-stage links stage-local: prefer linking only to summaries or knowledge notes that are already meaningful by the end of this stage. Avoid relying on `home.md` or other shape-stage routes as the main navigation surface.
- Prefer direct file-reading and search tools over shell commands for routine file inspection.

## Notes

- Bias structure toward canonical entities, claims, timelines, and stable indexes.
- Use taxonomy and ontology only as means to improve retrieval, navigation, and evidence tracking.
- For small Sources, prefer a minimal stable substrate over exhaustive flow sprawl.
- When you add wikilinks, target real Context Graph notes by exact basename or explicit relative path. Do not invent title-style links unless that exact title is also a declared note label or alias.
- Do not add wikilinks in structure outputs to files that the shape stage creates later. If a route belongs in `home.md` or a later shaped note, leave plain text now and let the later stage add the link.
- Build task-aware claims, entities, topics, and indexes from the summaries and their source refs. Use clear filenames such as `knowledge/rent-growth-claims.md`, `knowledge/market-entities.md`, or `knowledge/index.md`.
- For summary references, prefer explicit links like `[[summaries/<source-name>/summary]]`, `[[summaries/<source-name>/manifest]]`, page note links, or plain source refs. For knowledge notes, prefer the final filename stem under `knowledge/`.

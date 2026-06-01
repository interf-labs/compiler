# Summarize

Turn source files into source-named coverage folders with source metadata.

Contract type: `build-file-evidence`

## Requirements

- Each summary must start with literal JSON frontmatter between `---` lines, not a fenced code block.
- Required opening shape:
- `---`
- `{`
- `  "source_file_id": "file-example",`
- `  "source": "example.md",`
- `  "source_path": "example.md",`
- `  "source_locator": "agent-readable/source/ref",`
- `  "source_kind": "markdown",`
- `  "evidence_tier": "primary",`
- `  "truth_mode": "source-grounded",`
- `  "state": "complete"`
- `}`
- `---`
- Do not wrap that JSON in triple backticks or emit ```json anywhere in the summary file.
- Prefer direct file-reading and search tools over shell commands for routine file inspection.
- Include a clear abstract either in frontmatter or under a markdown `## Abstract` heading so a human can skim the summary quickly.
- Valid abstract forms for deterministic validation are:
- frontmatter key `"abstract"` with a real sentence, or
- a markdown heading `## Abstract` followed by at least one sentence
- A bare `Abstract` label without markdown heading syntax does not count.
- Do not skip the abstract just because the overview section is present.
- Build `summaries/` as a coverage layer:
  - one folder per source file, named exactly from `runtime/expected-inputs.json` metadata `source_path`
  - keep the original file extension in the folder name; do not strip `.md`, `.txt`, `.pdf`, or similar suffixes
  - non-paginated sources: `summaries/<source_path>/summary.md`, for example `summaries/01-source-note.md/summary.md`
  - PDFs, decks, images, or paginated/visual reports: `summaries/<source_path>/manifest.md` plus `pages/001/summary.md`, `pages/002/summary.md`, etc. for every page/slide that can be inspected
  - if a page/slide cannot be extracted, record `not_extracted_reason` in the manifest instead of silently skipping it
  - optional `screenshot.png` or visual trace paths only when useful and available
- Preserve links back to sources through the `source_file_id`, `source`, `source_path`, `source_locator`, page,
  section, table, chart, figure, or region references the source provides.
- Connect every summary INTO the graph web — a summary that no note links and that
  links no note is a free-floating island and fails the whole-graph
  `graph_notes_connected` gate. Source refs point DOWN to the original file; they
  are NOT graph-note links and do not connect a summary to the web. So in each
  summary's body add at least one `[[wikilink]]` to a sibling note in the graph:
  a `summaries/<index>` / section index you maintain for the group, a related
  summary, or `[[home]]`. If this stage writes a per-group or per-folder index
  note under `summaries/`, link each summary to its index and link the index back
  from `home.md` (the entrypoint stage may also route `home.md` into the coverage
  layer). The point is reachability: every summary must be reachable through the
  link web, not only through a source_ref.
- Do not copy source files into the Context Graph. Store source references.
- Treat this stage as common grounding. Capture what each Source contains and
  how to get back to it; leave task-specific synthesis to `knowledge` and
  `entrypoint` unless a source detail is needed to preserve coverage.
- Read `runtime/expected-inputs.json` and write `runtime/reviewed-inputs.json`
  with one decision for every required expected input before DONE.
- Mark an expected source unit `used` or `reviewed` when you summarized it.
  Mark it `missing`, `blocked`, or `not-relevant` only with a concrete reason.
- `runtime/reviewed-inputs.json` must use this shape:
- `{`
- `  "kind": "interf-stage-reviewed-inputs",`
- `  "version": 1,`
- `  "generated_at": "2026-01-01T00:00:00.000Z",`
- `  "stage_id": "summarize",`
- `  "reviewed": [`
- `    { "resource_id": "source-unit-example", "decision": "used", "source_refs": ["example.md"] }`
- `  ]`
- `}`

## Notes

- Favor conservative, source-grounded summaries that preserve evidence tiers and leave broader synthesis for later stages.
- Read `runtime/source-manifest.json` for the authoritative Source inventory, `runtime/stage-inputs.json` for the exact source references assigned to this stage, and `runtime/expected-inputs.json` for coverage decisions.
- Read `runtime/project.json` and `runtime/stage-contract.json` so you know the Project intent, but keep summaries broadly reusable source coverage rather than final task answers.
- For large reports or decks, capture page coverage, scope, headline evidence, and chart/table routing in one light pass. Prefer explicit page records over one loose file summary.
- One summary folder per Source file; this is the file-level COVERAGE proof — it
  proves every file was read, not that every fact was captured. Summaries do not
  need to answer the final task, and missing coverage is detectable at the file
  level only, never claimed at the fact level.
- Keep scratch extraction commands single-purpose and non-destructive.

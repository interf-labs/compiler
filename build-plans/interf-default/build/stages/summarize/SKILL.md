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
  - one folder per source file, named so a human can match it to the original source file
  - non-paginated sources: `summaries/<source-name>/summary.md`
  - PDFs, decks, images, or paginated/visual reports: `summaries/<source-name>/manifest.md` plus `pages/001/summary.md`, `pages/002/summary.md`, etc. for every page/slide that can be inspected
  - if a page/slide cannot be extracted, record `not_extracted_reason` in the manifest instead of silently skipping it
  - optional `screenshot.png` or visual trace paths only when useful and available
- Preserve links back to sources through the `source_file_id`, `source`, `source_path`, `source_locator`, page,
  section, table, chart, figure, or region references the source provides.
- Do not copy source files into the Context Graph. Store source references.

## Notes

- Favor conservative, source-grounded summaries that preserve evidence tiers and leave broader synthesis for later stages.
- Read `runtime/source-manifest.json` for the authoritative Source inventory and `runtime/stage-inputs.json` for the exact source references assigned to this stage.
- For large reports or decks, capture page coverage, scope, headline evidence, and chart/table routing in one light pass. Prefer explicit page records over one loose file summary.
- Summaries prove coverage; they do not need to answer the final task.
- Keep scratch extraction commands single-purpose and non-destructive.

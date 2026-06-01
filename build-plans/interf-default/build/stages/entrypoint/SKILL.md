# Entrypoint

Assemble the primary Context Graph entrypoint and task-specific entrypoint
notes around the saved Project intent and coverage.

Contract type: `build-entrypoint`

This stage is service-owned in the default local runtime. The local service
uses the same stage shell, expected inputs, reviewed inputs, output validation,
and StageManifest path, but writes deterministic routing notes from already
prepared summaries and knowledge instead of asking an agent to reinterpret the
Source.

## Requirements

- Read `runtime/project.json`, `runtime/stage-contract.json.project.intent`,
  `runtime/expected-inputs.json`, summaries, and knowledge notes before writing
  outputs.
- Write `runtime/reviewed-inputs.json` with one decision for every required
  expected knowledge note before DONE.
- `runtime/reviewed-inputs.json` must use this shape:
- `{`
- `  "kind": "interf-stage-reviewed-inputs",`
- `  "version": 1,`
- `  "generated_at": "2026-01-01T00:00:00.000Z",`
- `  "stage_id": "entrypoint",`
- `  "reviewed": [`
- `    { "resource_id": "knowledge-example", "decision": "used", "source_refs": ["knowledge/example.md"] }`
- `  ]`
- `}`
- Use Project intent plus coverage from summaries and knowledge to shape
  `home.md`, `artifacts/`, and supporting graph routes.
- Write `home.md` as the downstream agent's primary entrypoint. It must tell
  the next agent where to start, what source refs matter, and what caveats or
  source re-checks remain.
- Route into all three fixed layers: `summaries/` (coverage — every Source file
  read), `knowledge/` (the connected knowledge web — traverse/query),
  `artifacts/` (task handoffs). Do not introduce a new top-level folder; richer
  structure nests under `knowledge/`.
- Close the whole-graph connectivity floor (`graph_notes_connected`): EVERY note
  in the graph — including every summary, not only the ones a knowledge note
  happened to cite — must be reachable through the link web. A summary that no
  note links is a free-floating island and fails readiness. Make `home.md` (or a
  coverage-index note it links, e.g. `summaries/index.md`) route into the
  coverage layer so every summary has at least one inbound link. For a large
  coverage layer, prefer a `summaries/` index note that links each summary and
  link that index from `home.md`, rather than listing hundreds of summaries
  directly in `home.md`. The floor is connectedness, not a quota — one genuine
  inbound or outbound link per note is enough; never fabricate links to pass.
- Write at least one task-specific entrypoint note under `artifacts/` when it
  helps the agent continue from prepared context.
- Each task entrypoint note must start with JSON frontmatter containing non-empty `task`, `source_refs`, `handoff_type`, and `caveats`. Optional routing fields (for example a source-evidence posture or a coverage state) may be added when they help the next agent, but the gating keys are those four.
- `source_refs` must point to original Source locations, not just generated summaries or knowledge notes.
- Task entrypoint notes should tell the downstream agent which original Source files, pages, figures, tables, sections, units, periods, series, and caveats matter for the task.
- Task entrypoint notes should route the downstream agent to original Source refs for exact claims. Do not present generated summaries or knowledge notes as the dataset or source of truth.
- Do not leave the scaffold `Not yet built.` placeholder in place.
- When you add wikilinks, target real Context Graph notes by exact basename or explicit relative path.
- If you introduce a new note name in `home.md` or another shaped output, the same stage must also create that note file.
- Prefer direct file-reading and search tools over shell commands for routine file inspection.
- When a source value is approximate, preserve it as a bounded range and say why it is approximate.
- Do not invent finer precision than the source supports.
- For unlabeled charts, screenshots, diagrams, or visual marks, either create a measurement note with source-grounded visual reference, calibration, method, and uncertainty, or tell the downstream agent to inspect the original Source before answering.
- Keep approximate values consistent across `home.md`, focused indexes, and claim/entity notes.
- Do not copy expected answers into the Context Graph.

## Notes

- Use the saved Project intent and coverage metrics to bias the final Context Graph toward the job it should be especially good at.
- Do not copy benchmark expected answers into the final Context Graph.
- Prefer the saved summary evidence and structured notes when they already preserve routing, source refs, and caveats.
- Reopen source references during shaping when exact wording, table values, chart reads, or provenance-sensitive claims matter.
- If a prepared value depends on source-derived values, route the entrypoint or handoff to source refs and preserve source granularity instead of fabricating precision.
- Prefer better routing, prioritization, and focused navigation over speculative synthesis.
- Any wikilinks you add to `home.md` or indexes must resolve to real Context Graph note basenames or explicit relative paths.

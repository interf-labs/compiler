# Shape

Shape the task-specific Artifact handoff and final Context Graph index
around the saved task focus and Checks.

Contract type: `build-query-shape`

## Requirements

- Use the Build task focus plus saved Check text to shape `artifacts/` handoffs and supporting graph routes.
- Write at least one task-specific handoff note under `artifacts/`.
- Each Artifact handoff must start with JSON frontmatter containing non-empty `task`, `source_refs`, `handoff_type`, `truth_mode`, `verification_state`, and `caveats`.
- `source_refs` must point to original Source locations, not just generated summaries or knowledge notes.
- Artifact handoff notes should tell the downstream agent which original Source files, pages, figures, tables, sections, units, periods, series, and caveats matter for the task.
- Artifact handoff notes should route the downstream agent to original Source refs for exact claims. Do not present generated summaries or knowledge notes as the dataset or source of truth.
- Rewrite `home.md` into a real graph index note. Do not leave the scaffold `Not yet built.` placeholder in place, and do not make `home.md` the answer surface.
- When you add wikilinks, target real Context Graph notes by exact basename or explicit relative path.
- If you introduce a new note name in `home.md` or another shaped output, the same stage must also create that note file.
- Prefer direct file-reading and search tools over shell commands for routine file inspection.
- When a source value is approximate, preserve it as a bounded range and say why it is approximate.
- Do not invent finer precision than the source supports.
- For unlabeled charts, screenshots, diagrams, or visual marks, either create a measurement note with source-grounded visual reference, calibration, method, and uncertainty, or tell the downstream agent to inspect the original Source before answering.
- Keep approximate values consistent across `home.md`, focused indexes, and claim/entity notes.
- Do not copy expected answers into the Context Graph.

## Notes

- Use the saved task focus and Checks to bias the final Context Graph toward the job it should be especially good at.
- Do not copy benchmark expected answers into the final Context Graph.
- Prefer the saved summary evidence and structured notes when they already preserve routing, source refs, and caveats.
- Reopen source references during shaping when exact wording, table values, chart reads, or provenance-sensitive claims matter.
- If a saved Check depends on source-derived values, route the handoff to source refs and preserve source granularity instead of fabricating precision.
- Prefer better routing, prioritization, and focused navigation over speculative synthesis.
- Any wikilinks you add to `home.md` or indexes must resolve to real Context Graph note basenames or explicit relative paths.

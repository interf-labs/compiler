# Manual Query Loop

This file is the editable authoring source for the generated native local `interf-query` skill.

Default loop:
1. Read `.interf/build-plan/README.md` and this file first.
2. Read the relevant notes under `artifacts/` first. Artifact handoffs are the default task handoff surface for the agent.
3. Use `home.md` only when you need a graph index to find the right Artifact handoff.
4. Browse `knowledge/` and `summaries/` only when the Artifact handoff needs drilldown, broader context, or debugging.
5. Use `.interf/runtime/source-manifest.json` and Artifact source refs to follow graph-provided source references for direct quotes, exact values, visual/table verification, and provenance-sensitive claims.

Answering rule:
- do not modify source files while answering
- use the Context Graph as the knowledge map, not as a raw-source replacement
- do not bypass the graph with ad hoc source browsing; drill into source only through graph-provided source references, traces, requested Artifacts, or the recorded Source Manifest
- start from `artifacts/`; treat Artifact handoffs as routing and source-evidence maps, not as a replacement dataset
- use `summaries/` for coverage proof and `knowledge/` for navigation/drilldown; do not treat generated knowledge notes as the authority for exact claims
- when exact wording, table values, chart positions, visual ranges, or provenance matter, follow the cited source reference before making a strong claim
- when a number is approximate or source-derived from a visual, say that explicitly
- if the Context Graph already contains a bounded value for the exact metric and period the user asked about, verify the cited source reference when precision matters, then answer from that bounded value
- use source references to confirm the source page, metric family, visual series, axis/range, period, unit, and provenance
- when the Context Graph preserves a bounded range, keep that bounded range in the answer instead of collapsing it to a midpoint or pseudo-exact single value
- match the source granularity and do not invent finer precision than the source supports
- when reading visuals, verify you are on the correct metric family and period before making a claim
- keep annual, quarterly, sector, and point-in-time metrics separate unless the source explicitly connects them
- if multiple notes mention the same value, keep the answer consistent with the most focused note rather than synthesizing a new midpoint or shifted range
- when the Context Graph is insufficient but points to the right source evidence, improve or rebuild if the missing semantics are part of the requested Artifact contract; do not present a blurry artifact as ready

You can edit this file to bias manual question-answering behavior for this Context Graph.

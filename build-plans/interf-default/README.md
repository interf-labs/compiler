# Built-in Interf Build Plan

Built-in Build Plan that builds a three-layer Context Graph from source files:
summaries for coverage, knowledge for task-aware graph structure, and
artifacts for downstream agent handoff with links back to sources.

## Purpose

- General Build Plan implementation for preparing data for agents from source files.
- Build mixed source files into source coverage summaries, task-aware knowledge, Artifact handoffs, and links back to sources the downstream agent can use without rediscovering the Source.

## Artifacts

- `summaries` — source coverage directory at `summaries`
- `knowledge` — task-aware graph directory at `knowledge`; current default is
  flat `knowledge/*.md`, not required `knowledge/entities/`,
  `knowledge/claims/`, or `knowledge/indexes/` folders
- `artifacts` — task-specific agent handoff directory at `artifacts`
- `home` — Context Graph index file at `home.md`

## Stages

Each stage is a folder with a `SKILL.md` file. Interf creates the stage shell,
attaches source references and runtime context, runs the agent against that
plain-text Skill, and records evidence of what the stage wrote.

- `summarize` — Turn source files into source-named coverage folders with source metadata. (build-file-evidence; reads: none; writes: summaries)
- `structure` — Build the task-aware Context Graph structure from summaries, including entities, claims, topics, indexes, and links back to sources. (build-knowledge-structure; reads: summaries; writes: knowledge)
- `shape` — Shape task-specific Artifact handoffs and the final Context Graph around the saved task focus and Context Checks. (build-query-shape; reads: summaries, knowledge; writes: knowledge, artifacts, home)

## Why `home.md` exists here

This built-in Build Plan creates `home.md` as the Context Graph index.
Downstream agents should start from `artifacts/` for task handoffs and use
`home.md` only to navigate the graph. That is behavior of the `interf-default`
Build Plan implementation, not a local-service invariant.

This package is the built-in seed for `interf-default`.

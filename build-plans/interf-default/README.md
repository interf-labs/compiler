# Built-in Interf Build Plan

Built-in Build Plan that builds a three-layer Context Graph from source files:
summaries for coverage, knowledge for task-aware graph structure, and
home.md/artifacts for downstream agent entrypoints with links back to sources.

## Purpose

- General Build Plan implementation for preparing data for agents from source files.
- Build mixed source files into source coverage summaries, task-aware knowledge, entrypoints, and links back to sources the downstream agent can use without rediscovering the Source.

## Requested Outputs

- `summaries` — source coverage directory at `summaries`
- `knowledge` — task-aware graph directory at `knowledge`; current default is
  flat `knowledge/*.md`, not required `knowledge/entities/`,
  `knowledge/claims/`, or `knowledge/indexes/` folders
- `artifacts` — task-specific entrypoint note directory at `artifacts`
- `home` — Context Graph index file at `home.md`

## Stages

Each stage is a folder with a `SKILL.md` file. Interf creates the stage shell,
attaches source references and runtime context, runs the stage through the
runtime contract, and records evidence of what the stage wrote. Flexible stages
use the connected agent; `build-entrypoint` is assembled by the local service
from reviewed knowledge and summary resources so `home.md` is deterministic.

- `summarize` — Turn source files into source-named coverage folders with source metadata. (build-file-evidence; reads: none; writes: summaries)
- `knowledge` — Build the task-aware Context Graph knowledge layer from summaries, including entities, claims, topics, indexes, and links back to sources. (build-knowledge; reads: summaries; writes: knowledge)
- `entrypoint` — Assemble home.md as the primary agent entrypoint plus task-specific entrypoint notes around the saved Project intent and coverage. (build-entrypoint; reads: summaries, knowledge; writes: knowledge, artifacts, home)

## Why `home.md` exists here

This built-in Build Plan creates `home.md` as the primary Context Graph
entrypoint. Downstream agents should start from `home.md`, then follow links
into `knowledge/`, `summaries/`, `artifacts/`, and source refs. That is
behavior of the `interf-default` Build Plan implementation, not a local-service
invariant.

This package is the built-in seed for `interf-default`.

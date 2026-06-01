# Contributing

This public surface contains the files users and agent hosts need to inspect:
the README, license and security docs, bundled Skills, and public Build Plans.

Only files under `public-repo` are intended for this public surface. Do not add
source paths outside `public-repo`, maintainer docs, or repo-specific operating
instructions here.

The marketing website is maintained separately in `interf-labs/interf-website`.
Do not add landing-page code, website assets, or website operating notes to this
public SDK/package surface.

## What To Change Here

- Public product docs: the public README, `SECURITY.md`, and `TRADEMARKS.md`.
- Agent Skills: `skills/interf/`.
- Public Build Plans: `build-plans/`.

## Public Build Plans

Build Plans are inspectable folders. Keep them standalone:

- `build-plan.json` is the current technical filename for Build Plan stages,
  requested outputs, coverage requirements, and graph entrypoints.
- `build-plan.schema.json` is the current technical filename for the Build Plan
  package contract.
- `build/stages/<stage>/SKILL.md` contains stage instructions.
- `use/query/SKILL.md` tells agents how to read the Context Graph.
- `improve/SKILL.md` tells Interf how to revise the Build Plan when coverage,
  Stage Manifest validation, or graph output requirements fail.

For the shipped default Build Plan, edit `build-plans/interf-default/`.

## Agent Skills

Keep `skills/interf/SKILL.md` as the canonical Skill.

## Before Opening A PR

Check that public docs do not reference private paths such as internal source
trees, maintainer-only docs, or local machine paths.

If you changed a Build Plan, test it with an installed `interf`
runtime before submitting the change.

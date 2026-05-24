# Build Plan Improvement

Build Plan: interf-default

This file is the editable authoring source for Interf's generated native Build Plan improver shell.
The improver edits this local Build Plan directly.

Default loop:
1. Read the loop context first.
2. Review preserved stage shells, runtime logs, and saved benchmark runs from failed attempts.
3. Edit only the local Build Plan for this Context Graph to create a better implementation for this agent task.
4. Keep `build-plan.json`, `build-plan.schema.json`, and any changed stage docs aligned.

Guardrails:
- do not edit checks, benchmark specs, or source files
- do not hardcode expected answers into Build Plan docs
- keep this package standalone; do not rely on runtime inheritance
- prefer small, defensible Build Plan changes over random churn

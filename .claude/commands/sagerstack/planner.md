Invoke the `sagerstack-planner` skill.

This spawns a 4-member agent team (Researcher, Business Analyst, Solution Architect, Critical Analyst) to plan ONE phase from `docs/project-context.md`. Produces epics, user stories with FR/TR/AC, implementation plans, and critical analyses.

Requires `docs/project-context.md` to exist (created by `/sagerstack:code-planning`).

User input: $ARGUMENTS

If user input specifies a phase number (e.g., "1.1", "phase 2.3"), plan that phase.
If user input is "next", plan the next unplanned phase.
If user input is "manage", enter phase management mode (insert, remove, reorder, split, merge).
If user input is empty, present the phase list and ask which phase to plan.

**Flags:**
- `--skip-research` — Skip the research phase and architect web research. Researcher agent is not spawned. Solution Architect works from codebase + project-context only, without WebSearch/WebFetch. Use for straightforward epics with well-understood requirements.

Examples:
- `/sagerstack:planner 001` — Plan epic 001 with full research
- `/sagerstack:planner next --skip-research` — Plan next epic without research
- `/sagerstack:planner manage` — Manage epic structure

Invoke the `sagerstack-builder` skill.

This spawns a 2-member agent team (Software Developer, Code QA) to implement a planned epic. Executes implementation plans via TDD with QA validation gates per story.

Requires implementation plans to exist at `docs/phases/epic-{NNN}-{desc}/plans/` (created by `/sagerstack:planner`).

User input: $ARGUMENTS

If user input specifies an epic (e.g., "001", "epic-001-auth"), build that epic.
If user input includes "--story NNN", build only that story within the epic.
If user input includes "--continue", auto-chain to the next epic after completion.
If user input is "resume", resume from the last incomplete task.
If user input is "status", report current build progress.
If user input is empty, scan for planned epics and present build status.

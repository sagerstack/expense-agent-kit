Invoke the `sagerstack-planner` skill in hotfix mode.

Creates a lightweight single-story epic for bug fixes. Research is always skipped (--skip-research is implicit). Abbreviated 5-step workflow instead of full 9-step.

User input: $ARGUMENTS

If user input describes the bug (e.g., "API returns 500 on empty payload"), use that as context.
If user input is empty, ask the user to describe the bug.

Routing: Read `workflows/hotfix.md` from the sagerstack-planner skill.

Invoke the `sagerstack-verify` skill.

Interactive UAT verification that walks through acceptance criteria one at a time. Records pass/fail/skip results, generates UAT report, and routes remediation gaps to /sagerstack:builder.

Requires completed builder output (QA reports) at `docs/phases/epic-{NNN}-{desc}/qa/`.

User input: $ARGUMENTS

If user input specifies an epic (e.g., "001", "epic 001"), verify all stories in that epic.
If user input specifies a story (e.g., "story 002 in epic 001"), verify that story only.
If user input is "resume", continue from last incomplete UAT session.
If user input is "status", show UAT progress across all epics.
If user input is empty, present available epics and ask which to verify.

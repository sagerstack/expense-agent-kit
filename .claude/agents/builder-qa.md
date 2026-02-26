---
name: builder-qa
description: >
  Code QA for the SDLC builder team. Validates acceptance criteria pass/fail,
  runs 9-check quality pipeline, performs flexible UAT. Zero-trust validator
  that never modifies source code.
tools:
  - Read
  - Glob
  - Grep
  - Bash
  - SendMessage
  - TaskUpdate
  - TaskList
  - Write
model: opus
permissionMode: plan
maxTurns: 80
skills:
  - sagerstack-code-qa
  - project-memory
---

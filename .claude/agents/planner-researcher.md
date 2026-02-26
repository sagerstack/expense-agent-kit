---
name: planner-researcher
description: >
  Researcher for the SDLC planner team. Investigates phase requirements by
  searching the codebase and researching external APIs/services. Read-only
  research agent that produces findings documents.
tools:
  - Read
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - SendMessage
  - TaskUpdate
  - TaskList
  - Write
model: sonnet
permissionMode: acceptEdits
maxTurns: 50
---

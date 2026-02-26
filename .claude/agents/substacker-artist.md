---
name: substacker-artist
description: >
  Visual artist for the Substacker pipeline team. Generates hero images via AI,
  technical diagrams via Mermaid/D2, code screenshots via Carbonara API, and
  data visualizations. Records all prompts for reproducibility.
tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - WebFetch
  - ToolSearch
  - SendMessage
  - TaskUpdate
  - TaskList
model: sonnet
permissionMode: bypassPermissions
maxTurns: 60
skills:
  - substacker-images
---

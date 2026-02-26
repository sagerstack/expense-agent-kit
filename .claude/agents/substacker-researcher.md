---
name: substacker-researcher
description: >
  Researcher for the Substacker pipeline team. Discovers trending AI/ML topics
  across multiple sources and performs deep research to produce structured article
  outlines with source citations. Handles discovery and outline phases.
tools:
  - Read
  - Write
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - ToolSearch
  - SendMessage
  - TaskUpdate
  - TaskList
model: sonnet
permissionMode: bypassPermissions
maxTurns: 80
skills:
  - substacker-discover
  - substacker-outline
---

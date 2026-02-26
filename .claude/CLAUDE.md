# Agentic SDLC Framework

This repo contains a self-contained SDLC automation framework for Claude Code. It uses agent teams to plan and build software phase-by-phase with quality gates at every step.

## Pipeline Overview

The SDLC operates as a four-stage pipeline. Each stage must complete before the next begins.

```
/sagerstack:code-planning  -->  /sagerstack:planner  -->  /sagerstack:builder  -->  /sagerstack:verify
     (solo)                      (4-agent team)            (2-agent team)            (solo, interactive)

Produces:                   Produces:                           Produces:                    Produces:
docs/project-context.md     docs/phases/epic-{NNN}-{desc}/      src/ + tests/ (code)         docs/phases/epic-{NNN}-{desc}/qa/
                              epic.md                            docs/phases/epic-{NNN}-{desc}/qa/  uat-report.md
                              stories/story-{NNN}-{desc}.md        story-{NNN}-{desc}-qa-report.md
                              plans/story-{NNN}-{desc}-plan.md
                              plans/story-{NNN}-{desc}-critical-analysis.md
                              research/findings.md
```

### Stage 1: Code Planning (`/sagerstack:code-planning`)

Solo interactive session. Produces `docs/project-context.md` containing:
- Objective and Key Results
- Milestones broken into epics
- Architecture decisions
- All decisions made during planning

**Invocation**: Run `/sagerstack:code-planning` in any project.

### Stage 2: Phase Planning (`/sagerstack:planner`)

Spawns a 4-agent team to plan ONE epic at a time from `docs/project-context.md`.

**Team members**:
| Role | Agent Type | Model | Purpose |
|------|-----------|-------|---------|
| Team Lead | main session | opus | Orchestrates workflow, user Q&A |
| Researcher | `planner-researcher` | sonnet | Investigates requirements, APIs, codebase |
| Business Analyst | `planner-ba` | opus | Creates epics and user stories with FR/TR/AC |
| Solution Architect | `planner-architect` | opus | Generates implementation plans with 100% coverage |
| Critical Analyst | `planner-critic` | opus | Reviews impl plans for alignment and best practices |

**Workflow**: Research -> Proposals -> User confirms direction -> Stories with FR/TR/AC -> User confirms -> Technical Q&A -> Implementation plans -> Critical review (max 2 cycles)

**User Q&A checkpoints**: 3 mandatory points where user confirms before proceeding.

**Flags**: `--skip-research` skips Researcher agent and Architect web research for straightforward epics.

**Invocation**: Run `/sagerstack:planner`, select an epic. Use `/sagerstack:hotfix` for bug-fix epics.

### Stage 3: Building (`/sagerstack:builder`)

Spawns a 2-agent team to implement ONE epic from its planning artifacts.

**Team members**:
| Role | Agent Type | Model | Purpose |
|------|-----------|-------|---------|
| Team Lead | main session | opus | Assigns tasks, manages remediation |
| Software Developer | `builder-developer` | sonnet | Implements via TDD (red-green-refactor) |
| Code QA | `builder-qa` | opus | Validates AC, runs 9-check quality pipeline, UAT |

**Workflow**: Read impl plans -> Create tasks -> Developer implements via TDD -> QA validates per story -> Targeted remediation if failures (max 2 retries) -> Epic complete

**QA pipeline (9 checks)**: test suite, coverage (>=90%), type checking (mypy strict), linting (ruff), formatting, security (bandit), Docker build, CHANGELOG, git status

**Invocation**: Run `/sagerstack:builder`, select an epic to build.

### Stage 4: Interactive UAT (`/sagerstack:verify`)

Solo interactive session. Walks the user through acceptance criteria one at a time after the builder completes.

- Presents each AC (Given/When/Then) individually, waits for pass/fail/skip
- Infers severity from AC context (never asks user to rate)
- Writes results incrementally to `uat-report.md` (supports resume)
- Routes failures to `/sagerstack:builder` for remediation

**Invocation**: Run `/sagerstack:verify`, select an epic or story to verify.

## File Structure

### Skills (`.claude/skills/`)

| Skill | Slash Command | Purpose |
|-------|--------------|---------|
| `sagerstack-code-planning` | `/sagerstack:code-planning` | Pre-code planning, produces project-context.md |
| `sagerstack-planner` | `/sagerstack:planner` | Phase planning with agent team |
| `sagerstack-builder` | `/sagerstack:builder` | Phase building with agent team |
| `sagerstack-code-qa` | (preloaded by builder-qa) | QA validation methodology |
| `sagerstack-software-engineering` | (preloaded by builder-developer) | Python architecture standards (Vertical Slice + DDD) |
| `sagerstack-local-testing` | (preloaded by builder-developer) | Testing infrastructure, Docker, pytest |
| `sagerstack-verify` | `/sagerstack:verify` | Interactive UAT verification, AC-driven |
| `sagerstack-deploy-aws` | `/sagerstack:deploy-aws` | AWS Terraform deployment |
| `project-memory` | (preloaded by multiple agents) | Cross-session knowledge in docs/project_notes/ |

### Agent Definitions (`.claude/agents/`)

| Agent | Tools | Skills Preloaded |
|-------|-------|-----------------|
| `planner-researcher` | Read, Glob, Grep, WebSearch, WebFetch, SendMessage, TaskUpdate, TaskList, Write | (none) |
| `planner-ba` | Read, Write, Edit, Glob, Grep, SendMessage, TaskUpdate, TaskList | sagerstack-code-planning, project-memory |
| `planner-architect` | Read, Write, Edit, Glob, Grep, WebSearch, WebFetch, SendMessage, TaskUpdate, TaskList | project-memory |
| `planner-critic` | Read, Glob, Grep, WebSearch, WebFetch, SendMessage, TaskUpdate, TaskList, Write | project-memory |
| `builder-developer` | Read, Write, Edit, Glob, Grep, Bash, SendMessage, TaskUpdate, TaskList | sagerstack-software-engineering, sagerstack-local-testing, project-memory |
| `builder-qa` | Read, Glob, Grep, Bash, SendMessage, TaskUpdate, TaskList, Write | sagerstack-code-qa, project-memory |

### Slash Commands (`.claude/commands/sagerstack/`)

| Command | Invokes Skill |
|---------|--------------|
| `code-planning.md` | `sagerstack-code-planning` |
| `planner.md` | `sagerstack-planner` |
| `builder.md` | `sagerstack-builder` |
| `verify.md` | `sagerstack-verify` |
| `hotfix.md` | `sagerstack-planner` (hotfix mode) |

## Artifact Locations

When the SDLC runs against a target project, it produces artifacts in that project's `docs/` directory:

```
docs/
├── project-context.md              # From code-planning (Objective, Milestones, Key Results, Epics)
├── phases/
│   ├── epic-001-{desc}/
│   │   ├── epic.md                 # Epic with story portfolio
│   │   ├── research/
│   │   │   ├── findings.md         # Researcher output
│   │   │   └── story-001-{desc}-tech-research.md  # API/service research (if applicable)
│   │   ├── stories/
│   │   │   ├── story-001-{desc}.md          # User story with FR/TR/AC tables
│   │   │   └── story-002-{desc}.md
│   │   ├── plans/
│   │   │   ├── story-001-{desc}-plan.md     # Task-based implementation plan
│   │   │   ├── story-001-{desc}-critical-analysis.md  # Critical review
│   │   │   ├── story-002-{desc}-plan.md
│   │   │   └── story-002-{desc}-critical-analysis.md
│   │   └── qa/
│   │       ├── story-001-{desc}-qa-report.md  # QA validation report
│   │       ├── story-002-{desc}-qa-report.md
│   │       └── uat-report.md                  # Interactive UAT results (from /sagerstack:verify)
│   └── epic-002-{desc}/
│       └── ...
└── project_notes/                  # Project memory (cross-session)
    ├── bugs.md                     # Bug log with solutions
    ├── decisions.md                # Architectural Decision Records
    ├── key_facts.md                # Project configuration
    └── issues.md                   # Work history
```

## Quality Standards Enforced

These standards are embedded in the skills and automatically enforced by the agents:

- **Architecture**: Vertical Slice + DDD, strict domain purity (no infrastructure imports in domain)
- **Naming**: CamelCase everywhere (classes, functions, variables, tests)
- **Testing**: TDD mandatory (red-green-refactor), coverage >= 90%
- **Configuration**: No hardcoded values, all config from .env files
- **QA**: Zero-trust validation, 9-check pipeline, AC-driven testing
- **Planning**: 100% FR/TR/AC coverage mapping, cost flagging, max 2 refinement cycles
- **Git**: Feature branches per story, latest main merged before push

## Required Settings

The framework requires agent teams to be enabled in `.claude/settings.local.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "teammateMode": "in-process"
}
```

## Project Memory

Agents read and write cross-session knowledge to `docs/project_notes/`:

| File | Read By | Written By |
|------|---------|-----------|
| `bugs.md` | Critical Analyst, Developer, QA | Developer, QA |
| `decisions.md` | BA, Architect, Critical Analyst, Developer | Architect |
| `key_facts.md` | Architect, Developer | Architect, Developer |
| `issues.md` | BA | QA |

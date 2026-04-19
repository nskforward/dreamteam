# Dreamteam - Agent Development Framework

## Overview

Universal agent team for automated software development. Agents work sequentially through Master orchestrator, communicating via files.

**Quick start:** Give Master a high-level task → Master creates an Epic → Agents work through stages → Result delivered.

## Repository Structure

```
dreamteam/
├── .opencode/agents/           # Agent definitions (*.md files)
├── processes/                   # Workflow rules, conventions, limits
├── tasks/                       # Task templates and examples
├── epics/                       # Epic working directories
│   └── {id}-{name}/
│       ├── status.yml           # Epic state machine
│       ├── input.md             # Original user request
│       ├── artifacts/           # Epic-level artifacts
│       │   ├── requirements.md
│       │   ├── architecture.md
│       │   └── infrastructure.md
│       ├── workspace.md         # Shared working notes
│       ├── {stage-id}/          # Stage directory (e.g., 001-analysis)
│       │   ├── status.yml       # Stage metadata
│       │   └── conversation/
│       │       └── summary.md   # Agent communication summaries
├── src/                         # Final code (at project root, not in epics)
└── README.md
```

## Roles (7)

| Agent | Purpose | Stage |
|-------|---------|-------|
| `master` | Orchestrator. Creates epics, delegates tasks, updates status. Does NOT write code. | All |
| `analyst` | Requirements analysis, clarifies with business | analysis |
| `architect` | Technical design, infrastructure (requires user approval) | architecture |
| `developer` | Code implementation | development |
| `reviewer` | Code review, requests fixes until APPROVE | testing (review phase) |
| `tester` | Writes and runs tests, loops until all pass | testing (test phase) |
| `devops` | Infrastructure, participates in architecture stage | architecture |

## Workflow Types

| Type | Stages | Description |
|------|--------|-------------|
| `feature` | analysis → architecture → development → testing → done | Full development workflow |
| `bug` | analysis → architecture → development → testing → done | Bug fix workflow |

**Note:** The testing stage name is `testing` (not `test`).

## Epic Lifecycle

### 1. User → Master: New Task

User gives Master a high-level task description. Master determines workflow type and creates an epic.

### 2. Epic Creation

Master creates folder `epics/{number}-{short-name}/` with:
- `status.yml` — epic state machine
- `input.md` — original user request (saved for history)
- `artifacts/` — epic-level artifacts folder
- `workspace.md` — shared working notes
- Stage directories as work progresses

**Before creating a new epic, Master MUST ask for user confirmation** with proposed name.

### 3. Stage Progression

Each stage follows this pattern:
1. Master creates stage folder with `status.yml`
2. Master delegates to sub-agent via Task tool, passing input data
3. Sub-agent works, writes artifacts to epic-level `artifacts/`
4. Sub-agent writes execution summary to stage `conversation/summary.md`
5. Sub-agent provides structured response to Master
6. Master validates: positive keywords = success, failure keywords = NOT success
7. Master updates `status.yml`
8. For architecture stage: user approval required before proceeding

**How agents receive input:** Master provides the task context either directly in the system prompt or by specifying the path to `epics/{id}/input.md`. Agents do not read files on their own — all input is passed by Master when invoking the agent.

### 4. Testing Stage (Special Case)

The `testing` stage runs in two phases:
1. **Review phase:** reviewer → developer loop until reviewer APPROVE
2. **Test phase:** tester → developer loop until all tests pass

Communication is always through Master (sequential). User approval is NOT required in this stage.

## Epic status.yml Format

```yaml
workflow_type: feature
current_stage: architecture
stages:
  analysis:     { status: approved, approver: user, date: "2026-04-19T10:30:00Z" }
  architecture: { status: pending_approval, approver: user }
  development:  { status: pending }
  testing:      { status: pending }
```

**Stage statuses:**
- `pending` — not started
- `in_progress` — active work
- `pending_approval` — awaiting user approval (architecture stage only)
- `approved` — completed and approved
- `done` — epic completed

## Stage Transition Logic

### Architecture Stage
1. Architect works → `status: in_progress`
2. Architect completes → `status: pending_approval`
3. User approves → `status: approved` + `approver: user` + `date: ...`
4. If user rejects → `status: in_progress` + comment → Architect reworks

### Other Stages
1. Agent works → `status: in_progress`
2. Agent completes → `status: approved`

## User Approval Flow

1. User tells Master: "approve" or "approved" (any approving text)
2. Master updates `status.yml`: `approver: user, date: "...", status: approved`
3. Master proceeds to next stage

## Agent Communication

- `user → Master → sub-agent → Master → sub-agent → ... → Master → user`
- Agent-to-agent ONLY via Master (sequential)
- **Communication with user ALWAYS via Master** — agents never contact user directly
- This allows Master to track and display task status (e.g., "blocked — awaiting user clarification")
- History stored in `conversation/summary.md` of each stage

### Conversation Summary

Agents write execution summaries to `conversation/summary.md` after each run. Each summary should be brief (1-3 sentences) describing:
- What was accomplished in this execution
- Any issues encountered
- Next steps needed

Master stores these summaries to maintain context between stages and to track the workflow history.

### Devops Involvement

Devops participates only when explicitly requested by Master during the architecture stage. Architect does not call Devops directly — Architect notifies Master that Devops consultation is needed, and Master decides whether to invoke Devops as a separate agent.

## Validation Rules

Sub-agent MUST provide structured response on completion:
- Status: SUCCESS if positive, FAILED if contains failure keywords
- Success keywords: "done", "completed"
- Failure keywords: "error", "failed", "exception"
- All sub-agents must use format:
  ```
  Status: [SUCCESS/FAILED]
  Summary: <brief description>
  Next action: <what to do next>
  ```

## Naming Conventions

### Epic numbering
- Auto-incremented: Master finds the highest `{number}` in `epics/` and adds 1
- Format: `epics/{number}-{short-name}/`
- Master proposes name before creation, user can correct

### Stage numbering
- Auto-incremented inside epic: Master finds highest `{number}` in `{epic-id}/` and adds 1
- Independent from epic number (epic 003 can have stages 001-analysis, 002-architecture, etc.)
- Format: `{number}-{stage-name}`

## Status Files

### Epic-level status.yml
- Location: `epics/{id}-{name}/status.yml`
- Purpose: Main epic state machine — tracks `current_stage` and status of all stages

### Stage-level status.yml
- Location: `epics/{id}-{name}/{stage-id}/status.yml`
- Purpose: Stage metadata (status only, no actor field)
- Created: By Master, when stage folder is created

### Relationship
- Epic status.yml → global epic state
- Stage status.yml → local to that stage
- **Master creates and updates ALL status.yml files** — no other agent modifies them

### Folder Creation
- **Only Master creates stage folders** (`001-analysis/`, etc.)
- Master creates the folder before invoking the agent for that stage
- Folders are created progressively, not all at once at epic creation

## Permissions

- Write allowed ONLY in `epics/**`
- Master: `bash: false`
- Sub-agents: `bash: true` (can execute commands for build/test/deploy)
- Sub-agents called via Task tool

## Code Location

Final code is written directly to `src/` at project root (not inside epic folder). Workspace files (`workspace.md`) contain working notes only.

## Artifacts Location

All key artifacts are stored at epic level, not stage level:
- `epics/{id}/artifacts/requirements.md` — from analyst
- `epics/{id}/artifacts/architecture.md` — from architect
- `epics/{id}/artifacts/infrastructure.md` — from devops

## Dispute Resolution

**In ANY difficult or contentious situation: STOP the process and ask Master to request user assistance.** All agent disputes must be escalated to user via Master.

## Detailed Documentation

For detailed process descriptions, see:
- [`.opencode/agents/master.md`](.opencode/agents/master.md) — Master agent full specification
- [`.opencode/agents/analyst.md`](.opencode/agents/analyst.md) — Analyst agent
- [`.opencode/agents/architect.md`](.opencode/agents/architect.md) — Architect agent
- [`.opencode/agents/developer.md`](.opencode/agents/developer.md) — Developer agent
- [`.opencode/agents/reviewer.md`](.opencode/agents/reviewer.md) — Reviewer agent
- [`.opencode/agents/tester.md`](.opencode/agents/tester.md) — Tester agent
- [`.opencode/agents/devops.md`](.opencode/agents/devops.md) — Devops agent
- [`processes/workflow.md`](processes/workflow.md) — Detailed workflow stages
- [`processes/conventions.md`](processes/conventions.md) — Code conventions
- [`processes/limits.md`](processes/limits.md) — Process limits and rules
- [`tasks/task-template.md`](tasks/task-template.md) — Epic creation template
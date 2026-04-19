# Workflow

## Epic Types

| Type | Stages | Description |
|------|--------|-------------|
| `feature` | analysis → architecture → development → testing → done | Full development workflow |
| `bug` | analysis → architecture → development → testing → done | Bug fix workflow |

## Stage Details

### analysis
- Actor: analyst
- Input: epic description from user (via Master)
- Output: `artifacts/requirements.md` at epic level
- Requires: user clarification if needed (via Master)

### architecture
- Actors: architect, devops (consulted via Master)
- Input: epic description from Master
- Output: `artifacts/architecture.md`, `artifacts/infrastructure.md` at epic level
- Requires: user APPROVE before proceeding to development
- Note: Architect notifies Master if devops consultation is needed — Master decides whether to invoke devops

### development
- Actor: developer
- Input: `artifacts/architecture.md`, `artifacts/requirements.md` from epic-level artifacts
- Output: code in `src/` at project root (not in epic folder)

### testing
- Actors: reviewer, tester (sequential phases, not parallel)
- Input: code from `src/`, architecture docs from epic-level artifacts
- Process:
  1. reviewer reviews code from `src/`
  2. developer fixes issues (loop until reviewer APPROVE)
  3. tester writes and runs tests
  4. developer fixes issues (loop until all tests pass)
  5. tester APPROVE

### done
- All stages completed
- Epic status updated to `done`

## State Machine

Each epic has `status.yml` at epic level (`epics/{id}-{name}/status.yml`):
```yaml
workflow_type: feature
current_stage: architecture
stages:
  analysis:     { status: approved, approver: user, date: "2026-04-19T10:30:00Z" }
  architecture: { status: pending_approval, approver: user }
  development:  { status: pending }
  testing:      { status: pending }
```

Each stage has its own `status.yml` at stage level (`epics/{id}-{name}/{stage-id}/status.yml`):
```yaml
stage: analysis
status: in_progress
```

Stage-level status.yml is created by Master when the stage folder is created.

## Stage Transition Logic

### Architecture Stage
1. Architect works → `status: in_progress`
2. Architect completes → `status: pending_approval`
3. User approves → `status: approved` + `approver: user` + `date: ISO8601`
4. If user rejects → `status: in_progress` + comment → Architect reworks

### Other Stages
1. Agent works → `status: in_progress`
2. Agent completes → `status: approved`

## Agent Communication
- All communication via Master (sequential)
- No direct agent-to-agent communication
- History stored in `conversation/summary.md` folder per stage

## Deploy
- Deploy is a separate task for devops agent, NOT an automatic stage
- Triggered ONLY if user explicitly requested in task description
- Master creates a new task for devops when needed

## Approval Points
- Architecture stage: requires user APPROVE before development starts
- Testing stage: automatic approval when reviewer APPROVE + tester APPROVE + all tests pass

## Notes
- Stage name `testing` (not `test")
- devops participates in architecture stage only when Master invokes them
- One epic active at a time per Master
- Only Master creates stage folders and status.yml files
- All artifacts stored at epic level (`epics/{id}/artifacts/`), not stage level
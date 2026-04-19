# Task Template

## Epic Creation Request

When creating a new epic, Master should:

1. Determine workflow type (`feature`, `bug`)
2. Propose epic name to user and wait for confirmation
3. Create epic folder structure:
   ```
   epics/{number}-{short-name}/
   ├── status.yml           # Epic-level state machine
   ├── input.md             # Original user request
   ├── artifacts/           # Epic-level artifacts
   │   ├── requirements.md
   │   ├── architecture.md
   │   └── infrastructure.md
   ├── workspace.md         # Shared working notes
   ├── {stage-id}/
   │   ├── status.yml       # Stage-level state
   │   └── conversation/
   │       └── summary.md
   ```
   Note: Stage folders are created as work progresses, not all at once.

4. Initialize `status.yml` at epic root level:
```yaml
workflow_type: feature
current_stage: analysis
stages:
  analysis:     { status: in_progress }
  architecture: { status: pending }
  development:  { status: pending }
  testing:      { status: pending }
```

5. Initialize `status.yml` at stage level (e.g., `001-analysis/status.yml`):
```yaml
stage: analysis
status: in_progress
```

6. Write original user request to `input.md` in epic folder (for history)

## Agent Delegation Format

When delegating to sub-agent via Task tool:
```
Task: <agent-name>
Epic: <epic-id>
Stage: <stage-id>
Context: <brief summary of what needs to be done>
```

## Approval Format

User approves by telling Master. Master updates `status.yml`:
```yaml
stages:
  architecture: { status: approved, approver: user, date: "2026-04-19T10:30:00Z" }
```

## Stage Naming
- Stage folders use format: `{number}-{stage-name}` (e.g., `001-analysis`, `002-architecture`)
- All stage names: analysis, architecture, development, testing (NOT "test"), done

## Testing Stage Notes
The testing stage has two phases:
1. **Review phase:** reviewer checks code from `src/`, developer fixes issues until reviewer APPROVE
2. **Test phase:** tester writes/runs tests in `src/`, developer fixes issues until all tests pass

Both phases communicate through Master.

## APPROVE Mechanism

When reviewer or tester gives APPROVE:
1. Agent communicates APPROVE to Master in conversation/direct message
2. Master updates `status.yml` for the stage with `{ status: approved, date: "..." }`
3. Master proceeds to next phase/stage

## Stage Transition Logic

### Architecture
1. Architect works → `status: in_progress`
2. Architect completes → `status: pending_approval`
3. User approves → `status: approved` + `approver: user` + `date: ISO8601`
4. If user rejects → `status: in_progress` + comment → Architect reworks

### Others
1. Agent works → `status: in_progress`
2. Agent completes → `status: approved`
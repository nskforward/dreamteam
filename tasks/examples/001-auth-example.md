# Example Epic: User Authentication

## User Request
"Implement user authentication with JWT tokens for our API"

## Type: feature

## Epic Structure Created
```
epics/001-auth/
├── status.yml
├── input.md
├── artifacts/
│   ├── requirements.md    (produced by analyst)
│   ├── architecture.md    (produced by architect)
│   └── infrastructure.md  (produced by devops)
├── workspace.md
├── 001-analysis/
│   ├── status.yml
│   └── conversation/
│       └── summary.md
├── 002-architecture/
│   ├── status.yml
│   └── conversation/
│       └── summary.md
├── 003-development/
│   ├── status.yml
│   └── conversation/
│       └── summary.md
└── 004-testing/
    ├── status.yml
    └── conversation/
        └── summary.md
```

## Workflow Stages

1. **analysis** → analyst produces `artifacts/requirements.md`
2. **architecture** → architect + devops produce `artifacts/architecture.md` + `artifacts/infrastructure.md`, user approves
3. **development** → developer writes code to `src/`
4. **testing** → reviewer reviews until APPROVE, then tester runs tests until all pass

## User Approval Points
- After architecture (before development starts)

## Code Location
- Final code: `src/` at project root
- Tests: `src/tests/` or alongside source code

## Status After Architecture Approval
```yaml
workflow_type: feature
current_stage: development
stages:
  analysis:     { status: approved, approver: user, date: "2026-04-19T10:30:00Z" }
  architecture: { status: approved, approver: user, date: "2026-04-19T10:30:00Z" }
  development:  { status: in_progress }
  testing:      { status: pending }
```
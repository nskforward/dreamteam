# Dreamteam - Agent Development Framework

## Overview
Universal agent team for automated software development. Agents work sequentially through Master orchestrator, communicating via files.

## Repository Structure
```
dreamteam/
├── .opencode/agents/           # Agent definitions (*.md files)
├── processes/                   # Workflow rules (customizable per project)
├── tasks/                       # Task templates
├── epics/                       # Epic working directories
│   └── index.yml               # Global epic registry
└── README.md
```

**Per-epic structure:**
```
epics/{type}-{name}/
├── status.yml                  # Epic state machine
├── {stage-id}/                 # Task directory (e.g., 001-analysis)
│   ├── status.yml              # Task metadata
│   ├── artifacts/             # Final deliverables
│   ├── drafts/                 # Drafts, correspondence
│   └── conversation/           # Agent communication history
└── workspace/                  # Agent working files
    ├── analyst/
    ├── architect/
    ├── developer/
    ├── reviewer/
    ├── tester/
    └── devops/
```

## Roles (7)
| Agent | Purpose |
|-------|---------|
| `master` | Orchestrator. Creates epics, delegates tasks, updates status. Does NOT write code. |
| `analyst` | Requirements analysis, clarifies with business |
| `architect` | Technical design, infrastructure (requires approval) |
| `developer` | Code implementation |
| `reviewer` | Code review, checks quality |
| `tester` | Writing and running tests |
| `devops` | Infrastructure (participates in architecture stage) |

## Workflow Types
| Type | Stages |
|------|--------|
| `feature` | analysis → architecture → development → testing → done |
| `bug` | analysis → architecture → development → testing → done |
| `task` | in-progress → done |

**Note:** `testing` (not `test`) stage name.

## Master Agent (created)
File: `.opencode/agents/master.md`

**Key principles:**
- Only orchestrates, NEVER writes code
- All context between stages via FILES ONLY
- Agent communication ONLY through Master (sequential)
- Validates agent response on completion
- Agents MUST provide work summary
- Failure keywords in summary = NOT successful

**Validation rules:**
- Agent MUST provide short work summary
- If summary contains "error", "failed", "exception" → NOT successful
- If summary is positive ("done", "completed", "готово") → successful

**Process:**
1. Receive task from user
2. Determine type, create epic if new
3. Delegate to sub-agent via Task tool
4. Validate result (check text response)
5. Update status.yml
6. Repeat for next stage

**Communication:**
- Answer user questions about epic/task status
- All sub-agent status update requests → through Master

## Epic Status.yml Format
```yaml
workflow_type: feature
current_stage: architecture
stage_status: pending_approval
stages:
  analysis:    { status: approved, approver: user, date: "..." }
  architecture: { status: pending_approval }
  development:  { status: pending }
  testing:      { status: pending }
```

## Communication Flow
- `user → Master → sub-agent → Master → sub-agent → ... → Master → user`
- Agent-to-agent ONLY via Master
- History stored in `conversation/` folder of task

## Permissions
- Write allowed ONLY in `epics/**`
- Sub-agents called via Task tool (no bash)

## OpenCode Agent Format
```yaml
---
description: Agent description
mode: primary|subagent
permission:
  write:
    "*": deny
    "epics/**": allow
tools:
  write: true
  edit: true
---

# Role section
# Principles
# Workflow types
# Process
# Validation
# Communication with user
```

## TODO
- [ ] Create remaining 6 agents
- [ ] Define processes (workflow.md, conventions.md, limits.md)
- [ ] Create task templates
- [ ] Test workflow with first epic

## Pending Questions
- Sub-agent launch syntax (via Task tool) - TODO: verify

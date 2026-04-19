# Limits

## Agent Permissions

### Write Operations
- ALL agents: write allowed ONLY inside `epics/**`
- No write access outside epics folder

### Bash Operations
- Master: NO bash (no exceptions)
- Sub-agents: determined by each agent's definition in `.opencode/agents/*.md`

### Tools
- write: true for all sub-agents (in epics)
- edit: true for all sub-agents (in epics)
- bash: determined by each agent's definition

## Process Limits

### Epic
- One epic active at a time per Master
- Epic must complete (done) before new epic starts

### Review Cycles
- Unlimited iterations (developer ↔ reviewer)
- Until reviewer gives APPROVE

### Test Cycles
- Unlimited iterations (developer ↔ tester)
- Until all tests pass

### Timeouts
- No automatic timeouts
- User can interrupt at any point

## What Agents Must NOT Do

1. **Never modify status.yml directly** — only Master updates status files
2. **Never create stage folders** — only Master creates stage folders
3. **Never skip stages** (architecture approval is mandatory)
4. **Never deploy without explicit user request**
5. **Never directly communicate with other agents** (only via Master)
6. **Never write outside epics/ folder** (except final code to `src/` at project root)

## Stage Folder Creation
- **Only Master creates stage folders** (`001-analysis/`, etc.)
- Master creates the folder before invoking the agent for that stage
- Stage folders are created progressively, not all at once at epic creation

## Validation Rules

Agent must provide structured response on completion:
- Status: SUCCESS if positive, FAILED if contains failure keywords
- Success keywords: "done", "completed"
- Failure keywords: "error", "failed", "exception"

## Stage Names
- Stage for running tests is called `testing` (NOT "test")
- All stage names in status.yml must match: analysis, architecture, development, testing, done

## Artifact Requirements
- analysis: must produce `artifacts/requirements.md` at epic level
- architecture: must produce `artifacts/architecture.md` and `artifacts/infrastructure.md` at epic level
- development: must produce code in `src/` at project root
- testing: must produce test files and passing tests in `src/`

## New Epic Confirmation
**Master MUST ask for user confirmation before creating a new epic.** Propose name and wait for user response.

## Dispute Resolution
**In ANY difficult or contentious situation: STOP the process and ask Master to request user assistance.**
# Conventions

## Code Style

### General
- Follow language-specific best practices
- Use meaningful variable/function names
- Keep functions small (single responsibility)
- Comment complex logic

### Go
- Use `gofmt` for formatting
- Follow Go idioms (effective go)
- Error handling: wrap with context

### Rust
- Run `cargo fmt`
- Follow `clippy` recommendations
- Use Result for error handling

### JavaScript/TypeScript
- Use ESLint + Prettier
- Prefer const over let
- Use typed arrays/functions

### Python
- Follow PEP 8
- Use type hints
- Docstrings for public functions

## File Structure

```
epics/{id}-{name}/
├── status.yml
├── input.md
├── artifacts/
│   ├── requirements.md
│   ├── architecture.md
│   └── infrastructure.md
├── workspace.md
├── {stage-id}/
│   ├── status.yml
│   └── conversation/
│       └── summary.md
```

## Code Location

Final code is written directly to `src/` at project root (not inside epic folder). Workspace file (`workspace.md`) contains working notes only.

## Artifacts Naming

| Artifact | Location | Description |
|----------|----------|-------------|
| `requirements.md` | `epics/{id}/artifacts/` | Detailed requirements from analyst |
| `architecture.md` | `epics/{id}/artifacts/` | Technical architecture from architect |
| `infrastructure.md` | `epics/{id}/artifacts/` | Infrastructure decisions from devops |
| `summary.md` | `epics/{id}/{stage-id}/conversation/` | Agent communication summary |

## Naming Conventions
- Epic folders: `{number}-{short-name}` (e.g., `001-auth`, `002-payments`)
  - Number auto-incremented: Master finds highest `{number}` in `epics/` and adds 1
- Stage folders: `{number}-{stage-name}` (e.g., `001-analysis`, `002-architecture`, `003-development`, `004-testing`)
  - Number auto-incremented inside epic: Master finds highest `{number}` in `{epic-id}/` and adds 1
  - Independent from epic number
- Conversation logs: `summary.md` (single file per stage)

## Communication

### Conversation Summary
After each execution, agent writes a brief summary (1-3 sentences) to `conversation/summary.md` describing:
- What was accomplished
- Issues encountered
- Next steps needed

### Structured Response
All sub-agents must respond to Master in this format:
```
Status: [SUCCESS/FAILED]
Summary: <brief description>
Next action: <what to do next>
```

### Validation
- Success keywords: "done", "completed"
- Failure keywords: "error", "failed", "exception"
- Master validates by checking for these keywords in the response

## Testing Coverage
- Unit tests for core logic
- Integration tests for API/public interfaces
- Coverage target: see `processes/limits.md`

## Stage Names
- analysis
- architecture
- development
- testing (NOT "test")
- done

## Artifact Requirements
- analysis: must produce `artifacts/requirements.md` at epic level
- architecture: must produce `artifacts/architecture.md` and `artifacts/infrastructure.md` at epic level
- development: must produce code in `src/` at project root
- testing: must produce test files and passing tests

## New Epic Confirmation
Before creating a new epic, Master MUST:
1. Propose the epic name to user
2. Wait for user confirmation
3. User may suggest alternative name

## Dispute Resolution
In ANY contentious situation: STOP and ask Master to request user assistance via Master.
---
inclusion: always
---

# Orchestrator Workflow

## Role

The orchestrator is an agent, not a shell script. It reads task descriptions, reasons about them, and makes decisions — it does not blindly execute a loop.

## Task File Format

```yaml
# .auto-pilot/tasks.yaml
tasks:
  - id: "task-001"
    title: "Add input validation to registration form"
    description: |
      Validate email format and password strength on the user registration form.
      Acceptance criteria:
      - Email must match RFC 5322 format
      - Password must be at least 12 characters
      - Unit tests must cover valid and invalid inputs
    branch: "feature/task-001-registration-validation"
    depends_on: []
    status: pending   # pending | in-progress | done | failed

  - id: "task-002"
    title: "Add unit tests for AuthService"
    description: "Cover all public methods with unit tests, minimum 90% line coverage."
    branch: "feature/task-002-auth-service-tests"
    depends_on: ["task-001"]
    status: pending
```

## Orchestrator Decision Sequence

For each task with `status: pending`:

### 1. Understand the task
Read the description and acceptance criteria. If they are missing, vague, or contradictory — **stop and ask the human** before proceeding. Do not guess.

### 2. Check dependencies
If `depends_on` lists tasks that are not yet `done`, skip this task for now. Log: "Skipping task-002: depends on task-001 (status: in-progress)."

### 3. Evaluate complexity
Refuse tasks that exceed the complexity ceiling:
- Modifying authentication or authorization logic
- Breaking database schema changes
- Refactoring a core component used across many modules
- Tasks with no clear acceptance criteria
- Tasks touching security-sensitive configuration

If a task exceeds the ceiling, mark it `status: failed` with reason "Complexity ceiling exceeded — requires human implementation." and notify the user.

### 4. Create worktree and dispatch subagent

```bash
source .auto-pilot/.env
git worktree add .auto-pilot/task-001 -b feature/task-001-registration-validation $AUTOPILOT_BASE_BRANCH
```

Update `status: in-progress`. Dispatch a subagent with:
- Task description and acceptance criteria
- Worktree path: `.auto-pilot/task-001/`
- Branch name: `feature/task-001-registration-validation`
- Instructions to follow `subagent-execution.md`

### 5. On subagent completion
- Run quality checks on the worktree
- Push the branch: `git -C .auto-pilot/task-001 push $AUTOPILOT_REMOTE feature/task-001-registration-validation`
- Create PR/MR (see `pr-mr-creation.md`)
- Update `status: done`
- Remove worktree: `git worktree remove .auto-pilot/task-001`

### 6. On failure
- Update `status: failed`
- Log the full reason
- Continue to the next task — never abort the entire run

## Git Configuration Auto-Detection

```bash
# .auto-pilot/.env (override auto-detection)
AUTOPILOT_REMOTE=origin
AUTOPILOT_BASE_BRANCH=main
```

Auto-detection priority:
- **Remote**: `AUTOPILOT_REMOTE` env → single remote → fail with diagnostic
- **Branch**: `AUTOPILOT_BASE_BRANCH` env → `refs/remotes/<remote>/HEAD` → `main`/`master` probe → fail with diagnostic

## Logging

Append all orchestrator actions to `.auto-pilot/orchestrator.log`:

```
[2026-04-15 10:00:01] INFO  Starting run — 2 pending tasks
[2026-04-15 10:00:02] INFO  task-001: Creating worktree
[2026-04-15 10:00:05] INFO  task-001: Dispatching subagent
[2026-04-15 10:02:30] INFO  task-001: Quality checks passed
[2026-04-15 10:02:35] INFO  task-001: Branch pushed
[2026-04-15 10:02:38] INFO  task-001: PR created — https://github.com/org/repo/pull/42
[2026-04-15 10:02:39] INFO  task-001: Done
[2026-04-15 10:02:40] INFO  task-002: Skipped — depends on task-001 (in-progress)
[2026-04-15 10:02:40] INFO  Run complete. Done: 1, Failed: 0, Skipped: 1
```

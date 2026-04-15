---
inclusion: always
---

# Worktree Strategy

## Directory Structure

All worktrees MUST be created inside `.auto-pilot/` at the project root:

```
.
├── .auto-pilot/              # All agent worktrees (gitignored)
│   ├── .env                  # Remote and branch config
│   ├── tasks.yaml            # Task definitions and statuses
│   ├── orchestrator.log      # Execution log
│   ├── task-001/             # Worktree for task-001
│   ├── task-002/             # Worktree for task-002
│   └── task-003/             # Worktree for task-003
└── src/                      # Main working tree
```

`.auto-pilot/` must be in `.gitignore`. Worktrees are ephemeral — they are created for a task and removed after the PR/MR is created.

## Creating a Worktree

```bash
# Load config
source .auto-pilot/.env

# Create branch and worktree from base branch
git worktree add .auto-pilot/task-001 -b feature/task-001-short-description $AUTOPILOT_BASE_BRANCH
```

## Working Inside a Worktree

The worktree is a full copy of the repository on a new branch. The subagent works exclusively within its assigned path:

```bash
# All file edits happen inside the worktree
.auto-pilot/task-001/src/MyService.php
.auto-pilot/task-001/tests/MyServiceTest.php

# Commits are made from inside the worktree
cd .auto-pilot/task-001
git add src/MyService.php
git commit -m "Add MyService skeleton."
```

## Isolation Rules

- Subagents MUST only read and write files within their assigned worktree path
- Subagents MUST NOT access `.auto-pilot/task-002/` or any other worktree
- Subagents MUST NOT modify files in the main working tree
- If a task requires a shared resource (database, external API), use the same connection as the main tree — do not create a separate instance

## Cleaning Up

After the PR/MR is created:

```bash
git worktree remove .auto-pilot/task-001
```

If the worktree has uncommitted changes, force removal:

```bash
git worktree remove --force .auto-pilot/task-001
```

List all active worktrees:

```bash
git worktree list
```

## Dependency Detection

Before creating a worktree, check if the task depends on another in-flight task:

- If task B modifies the same files as task A (currently in-progress), task B must wait
- If task B depends on output from task A (e.g., a new service that task B will use), task B must wait
- If no overlap exists, tasks can run in parallel

When a dependency is discovered mid-execution, the subagent must stop, remove its worktree, and report the dependency to the orchestrator.

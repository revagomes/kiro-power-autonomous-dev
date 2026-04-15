---
description: Show current status of all auto-pilot tasks
---

Read `.auto-pilot/tasks.yaml` and report the current status of all tasks in a clear table.

For each task show: ID, title, status, branch, and any notes (e.g., PR/MR link if done, failure reason if failed).

Also report:
- Active worktrees: run `git worktree list` and show any `.auto-pilot/` entries
- Any tasks marked `in-progress` with no active worktree (possible stale state)

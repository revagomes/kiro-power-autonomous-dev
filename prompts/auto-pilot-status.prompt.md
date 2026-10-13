---
description: Show current status of all auto-pilot tasks
---

Read `.worktrees/tasks.yaml` and report the current status of all tasks in a clear table.

For each task show: ID, title, status, branch, and any notes (e.g., PR/MR link if done, failure reason if failed).

Call out `needs-pr` tasks prominently — these have a pushed branch but no PR/MR yet, and need a human to open the PR/MR manually.

Also report:
- Active worktrees: run `git worktree list` and show any `.worktrees/` entries
- Any tasks marked `in-progress` with no active worktree (possible stale state)

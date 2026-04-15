---
description: Remove all auto-pilot worktrees and reset task statuses to pending
---

Clean up all auto-pilot worktrees and optionally reset task statuses.

## Steps

1. Run `git worktree list` — identify all worktrees under `.auto-pilot/`
2. For each worktree found:
   - Check for uncommitted changes — warn the user if any exist before removing
   - Run `git worktree remove .auto-pilot/<task-id>` (or `--force` if confirmed by user)
3. Ask the user: "Reset all task statuses to `pending` in `tasks.yaml`? (yes/no)"
4. If yes, update `tasks.yaml` — set all `status` values to `pending`
5. Report what was cleaned up

## Safety check

Before removing any worktree with uncommitted changes, show the user what would be lost and ask for explicit confirmation. Never force-remove without confirmation.

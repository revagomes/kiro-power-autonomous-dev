---
description: Preview what auto-pilot would do — show execution plan without running anything
---

You are the auto-pilot orchestrator in dry-run mode. **Do not execute anything.** Read `.auto-pilot/tasks.yaml` and produce a clear execution plan.

## Your output

For each task, state:

**Would run:**
- Task ID and title
- Branch that would be created
- Reason it's eligible

**Would skip:**
- Task ID and title
- Reason: dependency not met / complexity ceiling / unclear acceptance criteria / already done/failed

**Execution order:**
- Show which tasks would run in parallel and which must run sequentially

**Questions before running:**
- List any tasks with missing or ambiguous acceptance criteria that need clarification before `/auto-pilot run`

End with: "Run `/auto-pilot run` to execute, or clarify the questions above first."

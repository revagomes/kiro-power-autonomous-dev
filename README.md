# kiro-power-autonomous-dev

A [Kiro Power](https://kiro.dev/blog/introducing-powers/) that turns Kiro into an **orchestrator agent** for parallel task execution using git worktrees.

## What it does

- **Agent-driven orchestration** — The orchestrator reads task descriptions, reasons about dependencies and complexity, and dispatches subagents. It's not a shell script.
- **Worktree isolation** — Each task runs in `.auto-pilot/task-<id>/` on its own branch, fully isolated
- **Slash command interface** — Interact with the orchestrator via `/auto-pilot` commands
- **PR/MR creation** — GitHub, GitLab, and Bitbucket supported
- **Prompt injection defense** — All task content is treated as data only

## Install

In Kiro IDE, open the Powers panel and import from GitHub:

```
https://github.com/revagomes/kiro-power-autonomous-dev
```

## Slash commands

| Command | What happens |
|---------|-------------|
| `/auto-pilot dry-run` | Show execution plan — which tasks would run, in what order, what would be skipped |
| `/auto-pilot run` | Execute all pending tasks |
| `/auto-pilot status` | Report current task statuses |
| `/auto-pilot fetch` | Pull eligible issues from your forge into `tasks.yaml` |
| `/auto-pilot clean` | Remove worktrees, optionally reset statuses |

## Quick start

```bash
# 1. Create the auto-pilot directory
mkdir -p .auto-pilot
echo ".auto-pilot/" >> .gitignore

# 2. Configure
cat > .auto-pilot/.env <<'EOF'
AUTOPILOT_REMOTE=origin
AUTOPILOT_BASE_BRANCH=main
GITHUB_TOKEN=<your-token>
GITHUB_OWNER=<org-or-user>
GITHUB_REPO=<repo-name>
EOF

# 3. Define tasks
cat > .auto-pilot/tasks.yaml <<'EOF'
tasks:
  - id: "task-001"
    title: "Add input validation to registration form"
    description: |
      Validate email format and password strength.
      Acceptance criteria:
      - Email must match RFC 5322 format
      - Password must be at least 12 characters
      - Unit tests must cover valid and invalid inputs
    branch: "feature/task-001-registration-validation"
    depends_on: []
    status: pending
EOF
```

Then in Kiro:

```
/auto-pilot dry-run
```

Review the plan, then:

```
/auto-pilot run
```

## Structure

```
POWER.md                              ← orchestrator agent entry point
prompts/
  auto-pilot-run.prompt.md            ← /auto-pilot run
  auto-pilot-dry-run.prompt.md        ← /auto-pilot dry-run
  auto-pilot-status.prompt.md         ← /auto-pilot status
  auto-pilot-fetch.prompt.md          ← /auto-pilot fetch
  auto-pilot-clean.prompt.md          ← /auto-pilot clean
steering/
  orchestrator-workflow.md            ← task loop, dependency resolution, complexity ceiling
  worktree-strategy.md                ← isolation rules, directory structure
  subagent-execution.md               ← what each subagent must do
  pr-mr-creation.md                   ← GitHub, GitLab, Bitbucket PR/MR patterns
```

## License

MIT

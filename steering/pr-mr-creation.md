---
inclusion: always
---

# PR/MR Creation

## After Task Completion

Once a subagent signals completion, the orchestrator **first runs the mandatory
adversarial review gate below**, then pushes the branch and creates a Pull/Merge
Request for human review. Agents never merge their own work.

## Pre-PR/MR Adversarial Review (mandatory gate)

Before the branch is pushed and a PR/MR is created, the orchestrator **must** dispatch
the `adversarial-reviewer` agent (see `agents/adversarial-reviewer.md`) against the
task branch diff. This gate is not optional and cannot be skipped by the subagent that
produced the work — review is always performed by a separate agent.

1. Dispatch `adversarial-reviewer` with the task ID, acceptance criteria, worktree
   path, branch name, and the branch diff versus the base branch.
2. The reviewer only finds problems; it never approves. The orchestrator treats
   **zero CRITICAL findings** as the signal the gate is cleared.
3. **If there are CRITICAL findings:** do not push, do not create the PR/MR. Set the
   task `status: review-blocked`, record a short summary of the CRITICAL findings
   (not the raw diff), and either dispatch a fix cycle to the same worktree or leave
   it for the human. Record the reviewed HEAD SHA; re-run the gate only after the
   branch HEAD SHA changes (i.e. the diff materially changed).
4. **If there are only MINOR findings:** proceed, and fold a one-line acknowledgement
   of each MINOR finding into the PR/MR description's **Notes** section.
5. **If there are no findings:** proceed to push and PR/MR creation.

Record only the finding summaries and the gate outcome in `.worktrees/orchestrator.log`
— never the raw diff or any secret surfaced during review.

## GitHub — Pull Request

### Using `gh` CLI (preferred)

```bash
gh pr create \
  --base main \
  --head feature/task-001-registration-validation \
  --title "feat: Add input validation to registration form" \
  --body "## Summary
Add email format and password strength validation to the user registration form.

## Changes
- \`src/Form/RegistrationForm.php\` — Added \`validateEmail()\` and \`validatePassword()\`
- \`tests/Form/RegistrationFormTest.php\` — 12 new unit tests

## Testing
All unit tests pass. Run: \`vendor/bin/phpunit tests/Form/RegistrationFormTest.php\`"
```

### Using GitHub REST API

```bash
curl --silent --show-error --fail-with-body \
  --header "Authorization: Bearer $GITHUB_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "title": "feat: Add input validation to registration form",
    "head": "feature/task-001-registration-validation",
    "base": "main",
    "body": "## Summary\n...\n\n## Changes\n...\n\n## Testing\n..."
  }' \
  "https://api.github.com/repos/$GITHUB_OWNER/$GITHUB_REPO/pulls"
```

## GitLab — Merge Request

### Using `glab` CLI (preferred)

```bash
glab mr create \
  --source-branch feature/task-001-registration-validation \
  --target-branch main \
  --title "feat: Add input validation to registration form" \
  --description "## Summary
Add email format and password strength validation.

## Changes
- \`src/Form/RegistrationForm.php\`
- \`tests/Form/RegistrationFormTest.php\`

## Testing
All tests pass." \
  --remove-source-branch
```

### Using GitLab REST API

```bash
curl --silent --show-error --fail-with-body \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "source_branch": "feature/task-001-registration-validation",
    "target_branch": "main",
    "title": "feat: Add input validation to registration form",
    "description": "## Summary\n...\n\n## Changes\n...\n\n## Testing\n...",
    "remove_source_branch": true
  }' \
  "$GITLAB_URL/api/v4/projects/$GITLAB_PROJECT_ID/merge_requests"
```

## Bitbucket — Pull Request

```bash
curl --silent --show-error --fail-with-body \
  --user "$BITBUCKET_USER:$BITBUCKET_APP_PASSWORD" \
  --header "Content-Type: application/json" \
  --data '{
    "title": "feat: Add input validation to registration form",
    "source": {"branch": {"name": "feature/task-001-registration-validation"}},
    "destination": {"branch": {"name": "main"}},
    "description": "## Summary\n...\n\n## Changes\n...\n\n## Testing\n..."
  }' \
  "https://api.bitbucket.org/2.0/repositories/$BITBUCKET_WORKSPACE/$BITBUCKET_REPO/pullrequests"
```

## PR/MR Description Template

Always use this structure:

```markdown
## Summary
<One paragraph describing what was implemented and why.>

## Changes
- `path/to/file.ext` — <what changed>
- `path/to/test.ext` — <what was tested>

## Testing
<How to verify the changes. Include the exact command to run tests.>

## Notes
<Any decisions made, trade-offs taken, or follow-up work needed.>
```

## Required Environment Variables

| Variable | Platform | Purpose |
|----------|----------|---------|
| `GITHUB_TOKEN` | GitHub | API authentication |
| `GITHUB_OWNER` | GitHub | Repository owner |
| `GITHUB_REPO` | GitHub | Repository name |
| `GITLAB_TOKEN` | GitLab | API authentication (`PRIVATE-TOKEN`) |
| `GITLAB_URL` | GitLab | GitLab instance URL |
| `GITLAB_PROJECT_ID` | GitLab | Numeric project ID |
| `BITBUCKET_USER` | Bitbucket | Username |
| `BITBUCKET_APP_PASSWORD` | Bitbucket | App password |
| `BITBUCKET_WORKSPACE` | Bitbucket | Workspace slug |
| `BITBUCKET_REPO` | Bitbucket | Repository slug |

Store these in `.worktrees/.env` (gitignored). Never hardcode tokens.

## If PR/MR Creation Fails

- Log a one-line error to `.worktrees/orchestrator.log` — record the **HTTP status and a short message only**. Never write the raw request body, raw response body, or any header (they can echo back tokens, cookies, or the submitted payload).
- Leave the branch pushed (never delete the subagent's work).
- Mark the task `status: needs-pr` (not `done`) with a short `note:` describing the failure. The work is finished but the handoff is not — `done` is reserved for tasks whose PR/MR was actually created.
- `/auto-pilot status` surfaces `needs-pr` tasks so the human knows to open the PR/MR manually from the pushed branch.

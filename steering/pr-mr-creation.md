---
inclusion: always
---

# PR/MR Creation

## After Task Completion

Once a subagent signals completion, the orchestrator pushes the branch and creates a Pull/Merge Request for human review. Agents never merge their own work.

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
curl --silent --fail \
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
curl --silent --fail \
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
curl --silent --fail \
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

Store these in `.auto-pilot/.env` (gitignored). Never hardcode tokens.

## If PR/MR Creation Fails

- Log the error to `.auto-pilot/orchestrator.log`
- Leave the branch pushed
- Mark the task `status: done` with a note that PR/MR creation failed
- The human can create the PR/MR manually from the pushed branch

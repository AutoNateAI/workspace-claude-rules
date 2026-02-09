# GitHub Flow

Complete workflow for interacting with GitHub using the CLI across both platform and clients repositories. **Includes Slack notification triggers.**

## When to Use

Attach this rule when:

- Picking up a new card
- Creating branches
- Managing PRs
- Updating project board
- Reviewing code
- Checking status

## Repository Context

| Repo | Path | GitHub | Default Branch |
|------|------|--------|----------------|
| Platform | `/Users/nathan.baker/code/fetch_workspace/platform` | `flockx-official/platform` | `staging` |
| Clients | `/Users/nathan.baker/code/fetch_workspace/clients` | `flockx-official/community-web-app` | `staging` |

**Project Board:** https://github.com/orgs/flockx-official/projects/8/views/35

---

## PR Reviewers

| Name | GitHub Username | Slack ID | Notes |
|------|-----------------|----------|-------|
| Kelsey Brennan | `kelseybrennan12` | U04CY6KMVC0 | Primary reviewer |
| Jonathan Chaffer | `jonathanchaffer` | U022PDG1LHH | Primary reviewer |
| Jared Currie | `jccurrie1` | U072ZF8Q292 | Primary reviewer |
| Meg Harr | `harrmegh` | U03D8EQ2C95 | Frontend reviewer |
| Devon Bleibtrey | `bleib1dj` | U04M3LQMWRJ | NOT in #fetch-devs Slack |

```bash
# Add all reviewers at once
gh pr edit <PR_NUMBER> --repo <REPO> --add-reviewer kelseybrennan12,jonathanchaffer,jccurrie1,harrmegh,bleib1dj
```

---

## Complete Workflow with Slack Triggers

### 1. Pick Up Card

```bash
# View issue details
gh issue view <ISSUE_NUMBER> --repo flockx-official/<REPO>
```

**Optional Slack:** Post to #fetch-devs if you want visibility:
```
Picking up #<NUMBER>: <title>
Branch: `feature/<NUMBER>-<description>`
```

### 2. Create Branch

```bash
cd /Users/nathan.baker/code/fetch_workspace/<repo>
git checkout staging && git pull
git checkout -b feature/<NUMBER>-<description>
```

### 3. Update Card with Progress

```bash
gh issue comment <ISSUE_NUMBER> --repo <REPO> --body "
## Progress Update - $(date +%Y-%m-%d)

**Status:** In Progress
**Branch:** \`feature/<NUMBER>-<description>\`

### Completed
- [x] Item 1

### In Progress
- [ ] Item 2
"
```

### 4. Create PR

```bash
gh pr create --repo <REPO> \
  --title "feat: Short description" \
  --body "## Summary
[Brief description]

## Related Issue
Closes #<ISSUE_NUMBER>

## Changes
- Change 1

## Testing
- [ ] Locally verified
" \
  --base staging

# Add reviewers immediately
gh pr edit <PR_NUMBER> --repo <REPO> --add-reviewer kelseybrennan12,jonathanchaffer,jccurrie1,harrmegh,bleib1dj
```

**Slack Notification - PR Ready for Review:**

```bash
source ~/.zshrc && curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_CLI_TOKEN" \
  -H "Content-type: application/json" \
  -d '{
    "channel": "C08C70B48PJ",
    "text": "PR ready for review:\n\n<https://github.com/flockx-official/REPO/pull/NUMBER|TYPE: TITLE>\n\nSUMMARY\n\nTests passing\nLocally verified\n\ncc <@U04CY6KMVC0> <@U022PDG1LHH> <@U072ZF8Q292> <@U03D8EQ2C95>"
  }'
```

### 5. Address Review Feedback

```bash
# View comments
gh api repos/flockx-official/<REPO>/pulls/<NUMBER>/comments \
  --jq '.[] | {author: .user.login, path: .path, line: .line, body: .body}'

# After fixing
git add . && git commit -m "fix: address review feedback" && git push

# Request re-review
gh pr edit <PR_NUMBER> --repo <REPO> --add-reviewer <REVIEWER>
```

### 6. Merge PR

```bash
gh pr merge <PR_NUMBER> --repo <REPO> --squash --delete-branch
```

**Optional Slack - Feature Shipped (for significant features):**

Post to #ao-planning for client visibility:
```
Shipped: #<CARD> - <title>

<1-2 sentence user-facing description>

PR: <https://github.com/flockx-official/REPO/pull/NUMBER|#NUMBER>
```

---

## Slack Notification Quick Reference

| Workflow Stage | Channel | Template |
|----------------|---------|----------|
| PR ready | #fetch-devs (C08C70B48PJ) | Tag reviewers, link PR |
| Blocked | #fetch-devs or DM | Tag person who can help |
| Feature shipped | #ao-planning (C08CETC7YS2) | For client visibility |
| Status update | #ao-planning | End of day / milestone |

**Reviewer Slack Tags (copy-paste):**
```
cc <@U04CY6KMVC0> <@U022PDG1LHH> <@U072ZF8Q292> <@U03D8EQ2C95>
```

**Note:** Devon (`<@U04M3LQMWRJ>`) is NOT in #fetch-devs - only tag on GitHub or in #ao-planning.

---

## Quick Reference Commands

### Status Check

```bash
# My open PRs
gh pr list --author="@me" --state=open --repo flockx-official/platform
gh pr list --author="@me" --state=open --repo flockx-official/community-web-app

# My assigned cards
gh project item-list 8 --owner flockx-official --format json --limit 100 | \
  jq '[.items[] | select(.assignees != null and (.assignees | index("nathanbaker-ao")))]'

# PR review status
gh pr view <NUMBER> --repo <REPO> --json reviewDecision,reviews,state
```

### PR Management

```bash
# View PR
gh pr view <NUMBER> --repo <REPO>

# Check CI status
gh pr checks <NUMBER> --repo <REPO>

# Mark ready (from draft)
gh pr ready <NUMBER> --repo <REPO>
```

### Code Review

```bash
# PRs to review
gh pr list --repo <REPO> --search "review-requested:@me"

# View diff
gh pr diff <NUMBER> --repo <REPO>

# Approve
gh pr review <NUMBER> --repo <REPO> --approve --body "LGTM!"

# Request changes
gh pr review <NUMBER> --repo <REPO> --request-changes --body "See inline comments"
```

#### Posting Inline Comments on Specific Lines

Use `gh api` to post comments directly on specific lines of code in a PR:

```bash
# Post an inline comment on a specific line
gh api repos/flockx-official/<REPO>/pulls/<NUMBER>/comments \
  --input - <<'EOF'
{
  "body": "Your comment text here",
  "commit_id": "<COMMIT_SHA_FROM_PR>",
  "path": "path/to/file.tsx",
  "line": 42
}
EOF
```

**Parameters:**
- `body`: The comment text (supports markdown)
- `commit_id`: The full SHA of the commit (find via `gh pr view <NUMBER> --json commits`)
- `path`: Relative path to the file (as shown in PR diff)
- `line`: Line number in the final file (the line you want to comment on)

**Example - Multiple inline comments:**

```bash
cd /Users/nathan.baker/code/fetch_workspace/<repo>

# Get commit SHA from PR
COMMIT=$(gh pr view 1234 --json commits --jq '.commits[0].oid')

# Comment 1: On line 210
gh api repos/flockx-official/<REPO>/pulls/1234/comments \
  --input - <<EOF
{
  "body": "⚠️ Architecture change: Friends are now user-scoped instead of twin-scoped.",
  "commit_id": "$COMMIT",
  "path": "apps/asi-one-native/src/components/personalization/connections/friend.tsx",
  "line": 210
}
EOF

# Comment 2: On line 173
gh api repos/flockx-official/<REPO>/pulls/1234/comments \
  --input - <<EOF
{
  "body": "Data migration: Old friends in PersonalizationContent won't be visible.",
  "commit_id": "$COMMIT",
  "path": "apps/asi-one-native/src/components/personalization/connections/friend.tsx",
  "line": 173
}
EOF
```

**Notes:**
- Comments appear inline in the PR diff view
- Use multiple `gh api` calls for multiple inline comments
- Each comment needs the full commit SHA
- Line numbers must exist in the diff for the comment to post
- Markdown formatting is supported (bold, italics, code blocks, etc.)

---

## Branch Naming

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/<issue>-<description>` | `feature/8270-message-sender` |
| Bug fix | `bugfix/<issue>-<description>` | `bugfix/3026-stream-loader` |
| Hotfix | `hotfix/<issue>-<description>` | `hotfix/9999-critical-fix` |

---

## Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:** `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `style`

**Examples:**
```bash
git commit -m "feat(chat): add participant drawer" -m "Closes #8168"
git commit -m "fix(chat): resolve StreamLoader race condition" -m "Fixes #3026"
```

---

## Handling Merge Conflicts

```bash
git fetch origin
git checkout feature/branch
git rebase origin/staging
# Fix conflicts
git add <resolved-files>
git rebase --continue
git push --force-with-lease
```

---

## Integration

- **slack-cli-integration.md** - Message templates and API patterns
- **team-directory.md** - Who to tag for what
- **morning-kickoff.md** - PR/card status overview
- **end-of-day-reflection.md** - Progress summaries
- **tdd-workflow.md** - Branch/commit flow

---

## Tips

1. **Always pull before branching**
2. **Small, focused PRs** - easier to review
3. **Link cards to PRs** - use "Closes #XXX"
4. **Add reviewers immediately** after creating PR
5. **Post to Slack** after creating PR for visibility
6. **Squash merge** for clean history
7. **Devon is NOT in #fetch-devs** - GitHub only or #ao-planning
8. **Inline comments**: Use `gh api repos/.../pulls/<N>/comments` to post comments on specific lines
9. **Multiple inline comments**: Get commit SHA once, then post all comments with that SHA

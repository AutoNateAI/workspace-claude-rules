# PR Submission Workflow

Standard workflow for submitting PRs after feature implementation and testing. Use when code is verified working and ready for review.

---

## Prerequisites

Before running this workflow:
- [ ] Code changes are complete
- [ ] TypeScript passes (`npx tsc --noEmit --skipLibCheck`)
- [ ] Manual testing confirms feature works as expected
- [ ] You're on a feature branch (not staging/main)

---

## The Workflow

### Step 1: Commit

```bash
git add -A && git commit -m "$(cat <<'EOF'
<type>(<scope>): <short description>

<longer description if needed>

EOF
)"
```

**Commit types:** `feat`, `fix`, `refactor`, `docs`, `chore`, `test`

**Example:**
```bash
git commit -m "$(cat <<'EOF'
feat(asi-one-native): implement chat with agent in planner task details

Wire up the chat button in Find Agents task detail to pre-fill the chat
input with @mention when tapped. Uses existing navigation pattern.

EOF
)"
```

---

### Step 2: Push

```bash
git push -u origin <branch-name>
```

---

### Step 3: Create PR

```bash
gh pr create \
  --title "<type>(<scope>): <description>" \
  --body "$(cat <<'EOF'
## Summary
- <bullet points of what changed>

## Previous Behavior
<What the code did before this PR. What was the user experience or system behavior?>

## Current Behavior
<What the code does after this PR. What changed from the user's or system's perspective?>

## Blast Radius
<What areas of the codebase are affected? Which components, screens, or flows touch this code? What is NOT affected?>

## Reasoning
<Why were these changes made at this level of the codebase (e.g., component vs. hook vs. dispatcher)? Why this approach over alternatives?>

## Test plan
- [ ] <manual test step 1>
- [ ] <manual test step 2>

EOF
)" \
  --base staging \
  --reviewer <github-username>
```

**Common reviewers:**
- `nhazekam` - Nick Hazekamp (mobile, primary reviewer)
- `sh-wallet` - Hemant Sharma (mobile, FlockX - tag for awareness)
- `kelseybrennan12` - Kelsey Brennan (planner, chat)
- `jonathanchaffer` - Jonathan Chaffer
- `jccurrie1` - Jared Currie (frontend)
- `harrmegh` - Meghan Harris (frontend)

**Mobile PRs:** Always add both `nhazekam` AND `sh-wallet` for awareness.

---

### Step 4: Post to Slack

Post a **one-liner** to both channels. Keep it short — no descriptions, no reviewer tags.

**#fetch-devs (C08C70B48PJ)** — No @mentions, just the link:

```bash
source ~/.zshrc && curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_CLI_TOKEN" \
  -H "Content-type: application/json" \
  -d '{
    "channel": "C08C70B48PJ",
    "text": "PR ready: <https://github.com/flockx-official/<repo>/pull/<number>|<title>>"
  }'
```

**#asi-one-mobile-app (C0999KNTC7M)** — For mobile PRs. @mention Nick Hazekamp and Hemant Sharma:

```bash
source ~/.zshrc && curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer $SLACK_CLI_TOKEN" \
  -H "Content-type: application/json" \
  -d '{
    "channel": "C0999KNTC7M",
    "text": "PR ready: <https://github.com/flockx-official/<repo>/pull/<number>|<title>> cc <@UNXP6CT6K> <@U050TLF65NU>"
  }'
```

---

### Step 5: Record Demo Video (if visual changes)

For PRs with UI changes, capture a simulator recording:

```bash
# Start recording
xcrun simctl io booted recordVideo /path/to/screenshots/pr-XXXX-feature-name.mov

# Perform the demo in simulator...

# Stop recording (Ctrl+C or from another terminal)
pkill -INT -f "recordVideo.*pr-XXXX"
```

Then drag-drop the video into the PR description's "Demo" section on GitHub web.

---

### Step 6: Update Session Log

Add entry to session log with:
- What was implemented
- What was tested
- Git workflow steps taken
- PR number and link
- Reviewer assigned

---

## Quick Reference

| Step | Command |
|------|---------|
| Commit | `git add -A && git commit -m "..."` |
| Push | `git push -u origin HEAD` |
| Create PR | `gh pr create --title "..." --body "..." --base staging --reviewer <user>` |
| Slack | `curl -s -X POST "https://slack.com/api/chat.postMessage" ...` |
| Update logs | Edit session-log.md |

---

## Example Full Flow

```bash
# 1. Commit
git add -A && git commit -m "feat(asi-one-native): implement view agent in planner task details"

# 2. Push
git push -u origin nathanbaker-ao/planner-task-detail-view-agent

# 3. Create PR
gh pr create \
  --title "feat(asi-one-native): implement view agent in planner task details" \
  --body "## Summary
- Wire up external link button to open agent profile in browser

## Previous Behavior
Tapping the external link icon on a planner task detail had no effect. The button was present but not wired up.

## Current Behavior
Tapping the external link icon opens the agent's profile page in the device browser using the agent's address URL.

## Blast Radius
Only affects the planner task detail screen (FindAgentsTaskDetail component). No other screens or flows are touched. The navigation utility used is shared but unchanged.

## Reasoning
The change lives in the task detail component because that's where the button and agent context already exist. No new hooks or services needed — the agent address is already available as a prop.

## Test plan
- [ ] Tap external link icon -> browser opens to agent profile" \
  --base staging \
  --reviewer nhazekam

# 4. Post to Slack (after getting PR number)
# PR ready: <PR_URL|title>

# 5. Update session log
```

---

## Related Rules

- github-flow.md - Full GitHub workflow including cards and branches
- session-logging.md - How to log session progress
- team-directory.md - Reviewer usernames and contact info

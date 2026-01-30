# Nathan's Fetch AI Workspace Rules

**Mission:** Solve 10+ problems per day across platform (Django) and clients (Next.js/React Native).

**Workspace Structure:**

- `clients/` - Frontend apps (fetch-llm, native, asi-one-native, agent-registry, web, business)
- `platform/` - Django backend with FastAPI, LangGraph, Celery
- `daily-notes/` - Structured notes by year/month/week/day
- `workspace-claude-rules/` - This consolidated rule set (single source of truth)

---

## Daily Workflow Rules

| Rule | When to Use | Trigger Phrases |
|------|-------------|-----------------|
| morning-kickoff.md | Start of day | "morning kickoff", "start my day", "what happened overnight" |
| session-logging.md | Throughout the day | "log this", "session note", "update session" |
| end-of-day-reflection.md | End of day | "wrap up", "end of day", "EOD reflection" |

---

## Research & Analysis Rules

| Rule | When to Use | Trigger Phrases |
|------|-------------|-----------------|
| research-workflow.md | Features, bugs, architecture | "research this", "analyze card", "investigate bug" |
| mermaid-diagrams.md | Any visualization need | "diagram this", "show flow", "visualize architecture" |

---

## Development Workflow Rules

| Rule | When to Use | Trigger Phrases |
|------|-------------|-----------------|
| tdd-workflow.md | Implementing features | "implement card", "TDD this", "test-first" |
| github-flow.md | Git/GitHub operations | "create branch", "open PR", "update card" |
| testing-conventions.md | Writing tests | "add test", "test structure" |
| pr-review-workflow.md | Reviewing others' PRs | "review PR", "help me review", "understand PR" |
| pr-feedback-workflow.md | Feedback on YOUR PRs | "check my PR", "fix my PR", "address comments" |
| pr-review-analysis.md | Analyze PR comments | "analyze PR feedback", "PR action plan" |
| pre-push-self-review.md | Before pushing code | "self review", "check my work", "pre-push review" |
| pr-submission-workflow.md | Submitting PRs | "submit PR", "create PR" |

---

## Communication Rules

| Rule | When to Use | Trigger Phrases |
|------|-------------|-----------------|
| slack-cli-integration.md | Slack interactions | "check slack", "post to channel", "read messages" |
| devon-meeting-updates.md | Bi-weekly Devon meetings | "devon update", "meeting prep" |
| team-directory.md | Finding contacts | "who handles", "contact for" |

---

## Organization Rules

| Rule | When to Use | Trigger Phrases |
|------|-------------|-----------------|
| daily-notes-structure.md | Creating notes | "create daily note", "note structure" |
| cursor-rules.md | Creating/editing rules | "new rule", "update rule" |
| self-improve.md | Pattern recognition | "add pattern", "rule improvement" |

---

## Stack-Specific Rules

### Platform (Django/Python)

- Django 5.2.8, FastAPI 0.115.14, Pydantic 2.11.7
- LangGraph 0.6.11 for agents
- PostgreSQL, Redis, Elasticsearch, Neo4j
- pytest, factory-boy, structlog

### Clients (Next.js/React)

- Next.js 15.1.6, React 19.x, TypeScript 5.8.2
- Hero UI, Tailwind 3.4.17, Framer Motion
- React Query, React Hook Form + Zod
- Cypress for all testing

---

## Quick Reference: GitHub CLI Commands

```bash
# FlockX Project Board
gh project item-list 8 --owner flockx-official --format json --limit 100

# My assigned cards
gh project item-list 8 --owner flockx-official --format json --limit 100 | jq '[.items[] | select(.assignees | index("nathanbaker-ao"))]'

# My PRs
gh pr list --author="@me" --state=all --limit=20 --repo flockx-official/platform
gh pr list --author="@me" --state=all --limit=20 --repo flockx-official/community-web-app

# Recent commits
git log --oneline --since="yesterday" --author="nathan" --pretty=format:"%h %s (%ad)" --date=short
```

---

## Daily Workflow Quick Start

### Starting the Day

```
"Morning kickoff - generate my daily report"
```

- Creates `daily-notes/YYYY/MM/week-NN/DD/morning-startup.md`
- Analyzes overnight GitHub activity
- Shows WIP cards and PRs
- Generates mermaid diagrams of current work

### During the Day

```
"Log: fixed the StreamLoader race condition in chat-messages.tsx"
```

- Appends to session log
- Updates related GitHub cards if mentioned

### Ending the Day

```
"End of day reflection"
```

- Creates `reflection.md` and `pickup-tomorrow.md`
- Summarizes commits, PRs, cards
- Prepares context for tomorrow

---

## Repository Links

- **Project Board:** https://github.com/orgs/flockx-official/projects/8/views/35
- **Platform:** https://github.com/flockx-official/platform
- **Clients:** https://github.com/flockx-official/community-web-app

---

## Integration Notes

- All rules assume `gh` CLI is authenticated
- Slack token at `$SLACK_CLI_TOKEN`
- Daily notes path: `/Users/nathan.baker/code/fetch_workspace/daily-notes/`
- Always use mermaid diagrams for flows and architecture
- Update GitHub cards with plans and thought processes
- Communicate efficiently in Slack threads

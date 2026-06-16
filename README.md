# focus-tasks

A personal priority task manager for Jen, powered by Claude (Cowork). Tasks are organized into a 4-quadrant priority matrix and automatically synced from Gmail, Google Calendar, and Slack.

## How it works

Claude runs a background sync every 2 hours (and on startup) that:

1. Pulls actionable emails from Gmail (starred, flagged, action keywords)
2. Pulls upcoming calendar events that need prep
3. Pulls Slack @mentions, saved messages, and reminders
4. Deduplicates against existing tasks
5. Appends new tasks to `tasks.json`
6. Pushes the updated file to this GitHub repo

The frontend (`index.html`) reads `tasks.json` and displays everything as a live Eisenhower matrix.

## Files

| File | Purpose |
|------|---------|
| `tasks.json` | Source of truth for all tasks |
| `index.html` | Frontend — priority matrix UI |
| `SKILL.md` | Instructions Claude follows for each sync |
| `sync-log.txt` | Append-only log of every sync run |
| `.env` | GitHub credentials (not committed) |

## Task format

```json
{
  "id": 13,
  "name": "Review GitHub security sign-in alert",
  "q": 1,
  "source": "gmail",
  "due": "Jun 16",
  "done": false,
  "isReminder": false
}
```

- `q` — quadrant: 1 (urgent + important), 2 (not urgent + important), 3 (urgent + not important), 4 (neither)
- `source` — `"gmail"`, `"calendar"`, `"slack"`, or `"manual"`
- `due` — human-readable date string or `null`
- `isReminder` — `true` only for Slack "Remind me later" items

## Quadrant rules

| Condition | Quadrant |
|-----------|----------|
| Due today or overdue | Q1 |
| Deadline within 3 days | Q1 |
| Deadline 4–14 days away | Q2 |
| Important, no deadline | Q2 |
| Urgent but low importance | Q3 |
| Slack reminder, no date | Q3 |
| Low urgency + low importance | Q4 |

## GitHub sync

`tasks.json` is pushed to this repo after every sync via the GitHub Contents API. Credentials are stored locally in `.env` (gitignored).

## Setup

1. Clone or create the repo on GitHub named `focus-tasks`
2. Add a `.env` file in the project folder:
   ```
   GITHUB_TOKEN=your_token_here
   GITHUB_USERNAME=your_username
   GITHUB_REPO=focus-tasks
   ```
3. Open the Cowork skill in Claude Desktop — syncs run automatically on schedule

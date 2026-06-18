# ⚡ Focus — Priority Task Manager

A personal task manager that automatically syncs from Gmail, Google Calendar, and Slack, and lets you triage and prioritize everything in one place.

**Live site:** https://jenniferendy.github.io/focus-tasks

---

## What it does

- **Inbox** — new tasks from Gmail, Slack, and Calendar land here first. You assign a priority with one click and they move to the dashboard.
- **Dashboard** — your full task list organized by urgency (Urgent → High → Medium → Low → N/A), with today's summary at the top.
- **Work Tracker** — a table view of all tasks with columns for Owner, Details, Urgency, Notes, and Due Date. Inline editable.
- **Auto-sync** — Cowork syncs your apps at 8am, 11am, 2pm, and 5pm daily and pushes updates to GitHub automatically.
- **Manual sync** — hit the "Sync now" button in the topbar anytime to trigger a sync outside the schedule.
- **Escalation flags** — tasks sitting unresolved for 5+ days get flagged so nothing falls through the cracks.

---

## How syncing works

Syncing is powered by **Cowork** in Claude Desktop. It reads `SKILL.md` for instructions, connects to Gmail, Google Calendar, and Slack, and writes new tasks to `tasks.json`.

**Sync schedule:** 8am, 11am, 2pm, 5pm daily (while Claude Desktop is open)

**To trigger a manual sync:** Click "Sync now" in the app, copy the message shown, and paste it into Cowork in Claude Desktop.

**VIP senders** (always high priority minimum):
- Jason Vottero — jasonv@endy.com / jason.vottero@sleepcountry.ca
- Melissa Bowie — melissa.bowie@dormezvous.com
- Jason Cosper — jason@endy.com

---

## File structure

```
focus-tasks/
├── index.html        # The app (front end)
├── tasks.json        # Task data — updated by Cowork on every sync
├── SKILL.md          # Cowork instructions and sync rules
├── .env              # GitHub token (never uploaded to GitHub)
└── sync-log.txt      # Log of every sync Cowork has run
```

---

## Pushing tasks.json to GitHub

Cowork updates `tasks.json` on your computer but can't push to GitHub directly due to a sandbox restriction. Run this PowerShell command after each sync to push manually:

```powershell
cd C:\Users\jenni\Documents\focus-tasks
$token = (Get-Content .env | Where-Object { $_ -match "GITHUB_TOKEN" }) -replace "GITHUB_TOKEN=",""
$content = [Convert]::ToBase64String([System.IO.File]::ReadAllBytes("C:\Users\jenni\Documents\focus-tasks\tasks.json"))
$headers = @{ Authorization = "Bearer $token"; "Content-Type" = "application/json" }
$sha = (Invoke-RestMethod -Uri "https://api.github.com/repos/jenniferendy/focus-tasks/contents/tasks.json" -Headers $headers -ErrorAction SilentlyContinue).sha
$body = @{ message = "sync: $(Get-Date -Format 'yyyy-MM-dd HH:mm')"; content = $content; sha = $sha } | ConvertTo-Json
Invoke-RestMethod -Method Put -Uri "https://api.github.com/repos/jenniferendy/focus-tasks/contents/tasks.json" -Headers $headers -Body $body
```

---

## Updating the app

When a new version of `index.html` is ready:

1. Drop the new `index.html` into `C:\Users\jenni\Documents\focus-tasks`
2. Go to github.com/jenniferendy/focus-tasks
3. Click `index.html` → pencil icon → select all → paste new code → Commit changes
4. Run the PowerShell push above to sync `tasks.json`
5. Wait ~60 seconds and refresh jenniferendy.github.io/focus-tasks

---

## Urgency levels

| Level | Meaning |
|---|---|
| 🔥 Urgent | Needs attention today — overdue or deadline within 3 days |
| ⬆ High | Important, deadline within 2 weeks or from a VIP |
| — Medium | Matters but not time-sensitive |
| ⬇ Low | Nice to do, no pressure |
| ✕ N/A | Logged for reference, no action needed |

---

## Escalation

Tasks at High or Medium priority that remain unresolved for 5+ days are automatically flagged with a ⚑ indicator. They don't move quadrants — you decide whether to escalate them.

---

## Built with

- HTML / CSS / JavaScript (single file, no framework)
- Hosted on GitHub Pages
- Synced via Cowork in Claude Desktop
- Integrations: Gmail, Google Calendar, Slack

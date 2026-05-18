---
name: project-status
description: Summarise ionstudioapps project progress from GitHub, Notion, and Slack. Use when asked for a project update, status report, standup summary, or "what's been happening".
---

# ionstudioapps Project Status

Pull the latest activity from GitHub, Notion, and Slack, then produce a single project progress summary.

## Required environment variables

The following must be set in the user's environment (from Rocky's `.env`):
- `NOTION_API_KEY`
- `NOTION_TASKS_DATABASE_ID`
- `SLACK_BOT_TOKEN`

GitHub uses the `gh` CLI (already authenticated).

Load credentials with:
```bash
source ~/.ionstudio/rocky.env 2>/dev/null || source ~/Projects/rocky/.env 2>/dev/null
```

---

## Step 1 — GitHub activity

Run these commands for each ionstudioapps repo:

```bash
# Recent commits across repos (last 7 days)
gh repo list ionstudioapps --limit 20 --json name --jq '.[].name' | while read repo; do
  echo "=== $repo ===";
  gh api repos/ionstudioapps/$repo/commits --paginate -q '.[].commit | "\(.author.date[:10]) \(.author.name): \(.message | split("\n")[0])"' 2>/dev/null | head -5;
done

# Open PRs
gh pr list --repo ionstudioapps/wheeltodo --state open --json title,author,createdAt,url 2>/dev/null
gh pr list --repo ionstudioapps/rocky --state open --json title,author,createdAt,url 2>/dev/null

# Open issues
gh issue list --repo ionstudioapps/wheeltodo --state open --json title,author,createdAt,labels 2>/dev/null
gh issue list --repo ionstudioapps/rocky --state open --json title,author,createdAt,labels 2>/dev/null
```

---

## Step 2 — Notion tasks

Query all open tasks from the Notion tasks database:

```bash
curl -s -X POST "https://api.notion.com/v1/databases/${NOTION_TASKS_DATABASE_ID}/query" \
  -H "Authorization: Bearer ${NOTION_API_KEY}" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "property": "Status",
      "status": { "does_not_equal": "Done" }
    },
    "sorts": [{ "property": "End date", "direction": "ascending" }]
  }'
```

Parse out: task title, status, assignee(s), due date, priority.

---

## Step 3 — Slack activity

Get the list of public channels, then fetch recent messages from project-related ones:

```bash
# List channels
curl -s "https://slack.com/api/conversations.list?limit=50&exclude_archived=true" \
  -H "Authorization: Bearer ${SLACK_BOT_TOKEN}" | jq '.channels[] | {id, name}'

# Get messages from relevant channels (last 48 hours)
# Use Unix timestamp: date -v-48H +%s (macOS) or date -d '48 hours ago' +%s (Linux)
SINCE=$(date -v-48H +%s 2>/dev/null || date -d '48 hours ago' +%s)

curl -s "https://slack.com/api/conversations.history?channel=CHANNEL_ID&oldest=${SINCE}&limit=30" \
  -H "Authorization: Bearer ${SLACK_BOT_TOKEN}"
```

Identify channels with names containing: `general`, `dev`, `eng`, `build`, `rocky`, `wheeltodo`, `product`, `design`.
Fetch messages from each relevant channel. Skip bot messages (where `bot_id` is set).

---

## Step 4 — Compile the summary

Produce a report in this format:

```
# ionstudioapps Project Status — [DATE]

## 🚀 Recent Shipping (GitHub — last 7 days)
- [repo] [date]: [commit message] — [author]
- Open PRs: [list or "none"]

## ✅ Tasks in Progress (Notion)
### By project / assignee:
- [task] — [assignee] — due [date] — [status]

### Overdue:
- [task] — [assignee] — was due [date]

## 💬 Team Pulse (Slack — last 48h)
### Key discussions:
- [channel] [author]: [summary of message thread]

## ⚠️ Blockers & Risks
- [anything that looks stuck, overdue, or flagged as blocked]

## 📋 Summary
[2-3 sentences: what's been done, what's in flight, any risks]
```

---

## Notes
- If a source fails (missing env var, API error), skip it and note it in the summary
- Focus on signal over noise: skip routine bot messages, merge commits, and closed items
- If asked for a specific project (e.g. "status of wheeltodo"), filter to that repo/tag only
- The Slack bot token may not have `search:read` scope — if `conversations.list` fails, skip Slack and note it

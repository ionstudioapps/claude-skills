---
name: schedule-meeting
description: Schedule a Google Meet from the terminal. Creates a Calendar event with a Meet link and optionally posts to Slack. Use when asked to schedule a meeting, set up a call, or create a Google Meet.
---

# Schedule Meeting

Create a Google Calendar event with a Google Meet link using Rocky's credentials.

## Credentials

Load credentials with:
```bash
source ~/.ionstudio/rocky.env 2>/dev/null || source ~/Projects/rocky/.env 2>/dev/null
```

Required vars from:
- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_REFRESH_TOKEN`
- `SLACK_BOT_TOKEN` (optional — for Slack notification)

## Step 1 — Resolve attendees from Notion

Before asking the user for emails, look up team members by name from the team accounts DB:

```bash
source ~/.ionstudio/rocky.env 2>/dev/null || source ~/Projects/rocky/.env 2>/dev/null

curl -s -X POST "https://api.notion.com/v1/databases/348e381c1b4780e3bab6ecd73bdae2f4/query" \
  -H "Authorization: Bearer ${NOTION_API_KEY}" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{}' | python3 -c "
import sys, json
data = json.load(sys.stdin)
for r in data.get('results', []):
    props = r['properties']
    name = props.get('Name', {}).get('title', [{}])
    gmail = props.get('Gmail', {}).get('email', None) or props.get('Gmail', {}).get('rich_text', [{}])
    name_val = name[0].get('plain_text','') if name else ''
    if isinstance(gmail, list):
        gmail = gmail[0].get('plain_text','') if gmail else None
    if name_val and name_val.lower() != 'rocky':
        print(f'{name_val}: {gmail}')
"
```

**Team directory:**
- Sua: alexsuakim@gmail.com
- Amithi: aliyanagamage@gmail.com
- Jade: lilykang0331@gmail.com
- Paoli: paolamarkart@gmail.com
- Johanna: johannaschwabe01@gmail.com

Match attendee names mentioned by the user (case-insensitive) to emails. If a name isn't in the list, ask the user for their email.

Always include the person who called the skill as an attendee. Detect them by running:

```bash
GH_USER=$(gh api user --jq '.login' 2>/dev/null)
echo "Current GitHub user: $GH_USER"
```

Then cross-reference with the Notion accounts DB — match `GitHub` field (which contains the GitHub URL or username) against `GH_USER`. Use their `Gmail` as their email. If no match is found, skip silently.

## Step 2 — Gather remaining details

If not provided in the request, ask for:
- **Title** — what is the meeting about?
- **Date & time** — when? (resolve relative times like "tomorrow 3pm" to ISO8601 in Asia/Seoul timezone)
- **Duration** — how long? (default: 1 hour)
- **Slack channel** — post the Meet link there? (optional)

## Step 3 — Get a Google access token

```bash
source ~/.ionstudio/rocky.env 2>/dev/null || source ~/Projects/rocky/.env 2>/dev/null

ACCESS_TOKEN=$(curl -s -X POST "https://oauth2.googleapis.com/token" \
  -d "client_id=${GOOGLE_CLIENT_ID}" \
  -d "client_secret=${GOOGLE_CLIENT_SECRET}" \
  -d "refresh_token=${GOOGLE_REFRESH_TOKEN}" \
  -d "grant_type=refresh_token" | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

echo "Token: $ACCESS_TOKEN"
```

## Step 4 — Create the Calendar event

```bash
# Build attendees JSON array from emails
# e.g. ATTENDEES='[{"email":"amithi@example.com"},{"email":"jade@example.com"}]'

curl -s -X POST \
  "https://www.googleapis.com/calendar/v3/calendars/primary/events?conferenceDataVersion=1" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "{
    \"summary\": \"TITLE\",
    \"description\": \"Scheduled via Claude Code\",
    \"start\": {\"dateTime\": \"START_ISO8601\", \"timeZone\": \"Asia/Seoul\"},
    \"end\": {\"dateTime\": \"END_ISO8601\", \"timeZone\": \"Asia/Seoul\"},
    \"attendees\": ATTENDEES,
    \"conferenceData\": {
      \"createRequest\": {\"requestId\": \"claude-$(date +%s)\"}
    }
  }" | python3 -c "
import sys, json
event = json.load(sys.stdin)
meet = event.get('hangoutLink', 'No Meet link generated')
title = event.get('summary', '')
start = event.get('start', {}).get('dateTime', '')
attendees = [a['email'] for a in event.get('attendees', [])]
print(f'✅ Meeting created!')
print(f'   Title: {title}')
print(f'   Time: {start}')
print(f'   Meet: {meet}')
print(f'   Attendees: {", ".join(attendees)}')
print(f'   EVENT_ID={event.get(\"id\",\"\")}')
print(f'   MEET_LINK={meet}')
"
```

## Step 5 — Post to Slack (if requested)

```bash
source ~/.ionstudio/rocky.env 2>/dev/null || source ~/Projects/rocky/.env 2>/dev/null

curl -s -X POST "https://slack.com/api/chat.postMessage" \
  -H "Authorization: Bearer ${SLACK_BOT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "{
    \"channel\": \"CHANNEL_ID_OR_NAME\",
    \"text\": \"📅 *TITLE*\n🕐 START_TIME\n👥 ATTENDEES\n🔗 MEET_LINK\"
  }"
```

## Output

After creating the meeting, always show:
- Meeting title and time
- Google Meet link (clickable)
- Attendee list
- Confirmation that Slack was notified (if applicable)

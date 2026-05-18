---
name: schedule-meeting
description: Schedule a Google Meet from the terminal. Creates a Calendar event with a Meet link and optionally posts to Slack. Use when asked to schedule a meeting, set up a call, or create a Google Meet.
---

# Schedule Meeting

Create a Google Calendar event with a Google Meet link using Rocky's credentials.

## Credentials

Load from Rocky's `.env` at `/Users/Sua/Projects/rocky/.env`:
- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_REFRESH_TOKEN`
- `SLACK_BOT_TOKEN` (optional — for Slack notification)

## Step 1 — Gather details

If not provided in the request, ask for:
- **Title** — what is the meeting about?
- **Date & time** — when? (resolve relative times like "tomorrow 3pm" to ISO8601 in Asia/Seoul timezone)
- **Duration** — how long? (default: 1 hour)
- **Attendees** — email addresses (check Rocky's Notion users DB or ask)
- **Slack channel** — post the Meet link there? (optional)

## Step 2 — Get a Google access token

```bash
source /Users/Sua/Projects/rocky/.env

ACCESS_TOKEN=$(curl -s -X POST "https://oauth2.googleapis.com/token" \
  -d "client_id=${GOOGLE_CLIENT_ID}" \
  -d "client_secret=${GOOGLE_CLIENT_SECRET}" \
  -d "refresh_token=${GOOGLE_REFRESH_TOKEN}" \
  -d "grant_type=refresh_token" | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

echo "Token: $ACCESS_TOKEN"
```

## Step 3 — Create the Calendar event

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

## Step 4 — Post to Slack (if requested)

```bash
source /Users/Sua/Projects/rocky/.env

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

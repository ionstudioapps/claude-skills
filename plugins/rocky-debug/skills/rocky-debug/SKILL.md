---
name: rocky-debug
description: Setup and debug Rocky bot — Slack/Notion/Google Calendar integrations. Use when Rocky isn't responding, env vars are missing, or setting up for a new teammate.
---

# Rocky Bot — Setup & Debug

Rocky is a team Slack bot that integrates Notion (tasks, users, memory, conversations) and Google Calendar. It runs on Node.js with the Anthropic API for intent routing.

## Required Environment Variables

All vars live in `.env` (copy from `env.example`):

| Variable | Where to get it |
|----------|----------------|
| `SLACK_BOT_TOKEN` | Slack app → OAuth & Permissions → Bot Token (xoxb-...) |
| `SLACK_SIGNING_SECRET` | Slack app → Basic Information → Signing Secret |
| `ANTHROPIC_API_KEY` | console.anthropic.com |
| `NOTION_API_KEY` | notion.so/my-integrations → your integration secret |
| `NOTION_TASKS_DATABASE_ID` | Notion DB URL → the ID after the workspace name |
| `NOTION_USERS_DATABASE_ID` | Same as above |
| `NOTION_MEMORY_DATABASE_ID` | Same as above |
| `NOTION_CONVERSATIONS_DATABASE_ID` | Same as above |
| `GOOGLE_CLIENT_ID` | Google Cloud Console → OAuth 2.0 credentials |
| `GOOGLE_CLIENT_SECRET` | Same as above |
| `GOOGLE_REFRESH_TOKEN` | Run `node scripts/google-auth.js` to generate |

## Common Issues

**Rocky not responding in Slack**
1. Check bot is invited to the channel: `/invite @Rocky`
2. Verify `SLACK_BOT_TOKEN` starts with `xoxb-`
3. Check the bot is running: `npm run dev`
4. Check Slack event subscriptions URL is pointing to the correct server

**Notion commands failing**
1. Confirm the integration is shared with all 4 databases (open DB → Share → add integration)
2. Check database IDs — get them from the URL: `notion.so/<workspace>/<DATABASE_ID>?v=...`

**Google Calendar not working**
1. Refresh token expires — re-run `node scripts/google-auth.js`
2. Check Google Calendar API is enabled in Google Cloud Console
3. Verify the calendar is shared with the Google account used for OAuth

**Starting fresh for a new teammate**
```bash
cp env.example .env
# Fill in all values
node scripts/google-auth.js   # generates GOOGLE_REFRESH_TOKEN
node scripts/setup-notion.js  # verifies Notion connection
npm run dev
```

## Architecture
- `index.js` → entry point, starts Slack bolt app
- `src/app.js` → Slack event handlers
- `src/handlers/intent/router.js` → routes messages to calendar/notion/general intents
- `src/integrations/` → Notion, Google Calendar, memory adapters
- `src/config.js` → validates and exports all env vars

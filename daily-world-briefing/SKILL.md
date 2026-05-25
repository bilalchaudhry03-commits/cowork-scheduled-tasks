---
name: daily-world-briefing
description: Daily morning world news briefing — top stories, S&P markets, and science/funding news via Slack DM
---

Send a daily world news briefing to your Slack DM.

## Setup — Customize These Values

Before using this task, replace the following placeholders:

| Placeholder | What to put here |
|---|---|
| `[YOUR_SLACK_USER_ID]` | Your Slack user ID (e.g. `U0ACAU48XQB`) — find it in your Slack profile |
| `[YOUR_SLACK_WORKSPACE]` | Your Slack workspace name (e.g. `myteamworkspace`) |
| `[YOUR_LOG_PATH]` | Local folder for saving daily briefing logs (e.g. `C:\Users\you\Documents\logs`) |

## Required Tools
- Slack MCP connector (`send_message` tool) — for delivery. Do NOT use ToolSearch to find it; use it directly.
- WebSearch — for fetching current news and market data

## Instructions

Search the web for today's top world news, S&P 500 market data, and science/funding news. Send ONE Slack DM to user `[YOUR_SLACK_USER_ID]`.

Use Slack mrkdwn formatting throughout. Do NOT use markdown `[text](url)` syntax — use `<url|text>` for all links.

## Searches to run
1. "world news today [current date]" — for top stories
2. "S&P 500 market today [current date]" — for market snapshot
3. "NIH NSF science funding news [current month year]" — for science policy

> **Tip:** Swap search #3 for any domain you care about, e.g. "AI research news", "climate policy news", "biotech funding news".

## Message format

```
🌍 *World Briefing — [Day, Date]*

*📈 Markets*
S&P 500: [value] ([%change]) | Dow: [value] ([%change]) | Nasdaq: [value] ([%change])
_[1-sentence driver: what's moving markets today]_

*🗞️ World News*
• [Story 1 — 1–2 sentences, include link if available as <url|title>]
• [Story 2]
• [Story 3]
• [Story 4]
• [Story 5]

*🔬 Science & Funding*
• [NIH/NSF/science policy item 1 — 1–2 sentences]
• [NIH/NSF/science policy item 2 if available]
_Nothing new today._ ← use this if no relevant science funding news found
```

## Rules
- World news: exactly 5 bullets, drawn from the last 24 hours. Prioritize geopolitical, major domestic US, and global health events. Skip celebrity/entertainment.
- Markets: report S&P, Dow, Nasdaq. One sentence on what's driving movement.
- Science & Funding: focus on NIH, NSF, federal research funding, university research policy, or major scientific announcements. If nothing in last 24h, extend to last 7 days. If truly nothing, say so.
- Keep each bullet to 1–2 sentences max — skimmable in under 2 minutes total.
- Use Slack mrkdwn: *bold*, _italic_, bullet •. No markdown headers (##). No [text](url) links.

## Save locally before sending
Write the briefing to a local file using PowerShell before sending:
```powershell
$date = Get-Date -Format "yyyy-MM-dd"
$outPath = "[YOUR_LOG_PATH]\world-briefing-$date.md"
$briefingContent | Set-Content -Path $outPath -Encoding utf8
```

## Failure handling
If Slack delivery fails, append a note to `[YOUR_LOG_PATH]\briefing-errors.log`:
```
$(Get-Date -Format 'yyyy-MM-dd HH:mm') — World briefing Slack delivery failed. Saved to $outPath
```

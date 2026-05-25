---
name: daily-paper-digest
description: Daily digest of new publications across your research topics, delivered via Slack DM with deduplication
---

Search for new papers across your research tracks and send a daily digest to your Slack DM. Tracks duplicates across runs so you never see the same paper twice.

## Setup — Customize These Values

Before using this task, replace the following placeholders:

| Placeholder | What to put here |
|---|---|
| `[YOUR_SLACK_USER_ID]` | Your Slack user ID (e.g. `U0ACAU48XQB`) — find it in your Slack profile |
| `[YOUR_SLACK_WORKSPACE]` | Your Slack workspace name |
| `[YOUR_LOG_PATH]` | Local folder for logs and the dedup file (e.g. `C:\Users\you\Documents\logs`) |

Then edit the **Search Topics** section below to match your research interests.

## Required Tools
- Slack MCP connector (`send_message` tool) — for delivering the digest. Do NOT use ToolSearch to find it; use it directly.
- bioRxiv MCP (`search_preprints`) — for preprint search
- PubMed MCP (`search_articles`, `get_article_metadata`) — for journal search
- Windows PowerShell — for reading/writing the dedup log

## Deduplication — MUST DO FIRST

Before searching, read the previously-seen papers log using PowerShell:
- Stable file path: `[YOUR_LOG_PATH]\paper-digest-seen.txt`
- Each line is a URL of a paper that has already appeared in a prior digest
- If the file does not exist, treat the seen list as empty

```powershell
$seenPath = "[YOUR_LOG_PATH]\paper-digest-seen.txt"
if (Test-Path $seenPath) { Get-Content $seenPath } else { Write-Host "NO_SEEN_FILE" }
```

After collecting all candidate papers, **exclude any whose URL is already in the seen list**.

## Search Topics

> **This is the main section to customize.** Replace the three tracks below with your own research areas.
> You can have fewer or more tracks — just update the Message Structure section to match.

**Track 1 — [YOUR TOPIC 1, e.g. "Chromatin Biology & Epigenetics"]**
Search bioRxiv, PubMed, and relevant journals for papers on:
- [Keyword / subtopic 1]
- [Keyword / subtopic 2]
- [Keyword / subtopic 3]

**Relevance filter for Track 1:** [Optional: describe what to include/exclude to keep results focused]

**Track 2 — [YOUR TOPIC 2, e.g. "Single-cell Genomics"]**
Search bioRxiv, PubMed, and relevant journals for papers on:
- [Keyword / subtopic 1]
- [Keyword / subtopic 2]
- [Keyword / subtopic 3]

**Track 3 — [YOUR TOPIC 3, e.g. "Machine Learning for Biology"]**
Search bioRxiv, PubMed, and relevant journals for papers on:
- [Keyword / subtopic 1]
- [Keyword / subtopic 2]
- [Keyword / subtopic 3]

Focus on papers published or posted in the last 48 hours. If the date window returns few results (e.g., API lag on bioRxiv), extend to the last 7–14 days and note this in the digest.

## Save Digest Locally First

Before sending to Slack, **always write the full formatted digest to a local file**:
```powershell
$date = Get-Date -Format "yyyy-MM-dd"
$outPath = "[YOUR_LOG_PATH]\paper-digest-$date.md"
$digestContent | Set-Content -Path $outPath -Encoding utf8
```
This ensures the digest is recoverable even if Slack delivery fails.

## Send via Slack MCP

Use the Slack MCP connector's `send_message` tool directly. Send three separate DMs to user `[YOUR_SLACK_USER_ID]` to stay under character limits.

Use Slack mrkdwn formatting. Structure each paper entry as:

`• <paper_url|Full Paper Title> (Journal/Source, Date) — one-sentence summary`

**CRITICAL LINK RULE:** The link text must be the COMPLETE paper title — never just the first word or a partial title. Use Slack's mrkdwn link syntax: `<url|full title here>`. Do NOT use markdown `[text](url)` — Slack does not render it.

Correct: `• <https://doi.org/10.1234/example|Karyopherins remodel the dynamic organization of the NPC transport barrier> (Nat Cell Bio, Apr 20) — high-speed AFM reveals how transport factors partition the FG barrier`

Incorrect: `• <https://doi.org/10.1234/example|Karyopherins> remodel the dynamic organization of the NPC transport barrier (Nat Cell Bio, Apr 20)`

## Message Structure

> Update the emoji, section names, and message count to match your tracks.

**Message 1 — Header + Track 1:**
```
📄 *Daily Paper Digest — [Date]*

*🧬 [Track 1 Name] ([N] papers)*
_Preprints_
• <url|Title> (bioRxiv, Date) — summary
...
_Publications_
• <url|Title> (Journal, Date) — summary
...
```

**Message 2 — Track 2:**
```
*🔬 [Track 2 Name] ([N] papers)*
_Preprints_
• <url|Title> (bioRxiv, Date) — summary
...
_Publications_
• <url|Title> (Journal, Date) — summary
...
```

**Message 3 — Track 3:**
```
*🤖 [Track 3 Name] ([N] papers)*
_Preprints_
• <url|Title> (bioRxiv, Date) — summary
...
_Publications_
• <url|Title> (Journal, Date) — summary
...
```

If no papers were found in a subsection, write `_Nothing new in the last 48 hours._`

Include a brief note at the end if you extended the date window beyond 48 hours.

## After Sending — Update Dedup Log

Only update the dedup log **after all three Slack messages have been confirmed sent**. If Slack delivery fails, do NOT update the log (so the papers will be retried next run).

Append each newly sent paper's URL (one per line):
```powershell
$seenPath = "[YOUR_LOG_PATH]\paper-digest-seen.txt"
$newUrls = @(
    "https://doi.org/...",
    "https://doi.org/..."
)
$newUrls | Add-Content -Path $seenPath -Encoding utf8
```

## Failure Handling

If the Slack MCP tool is unavailable or all send attempts fail:
1. The local digest file (already saved above) serves as the fallback record
2. Append a failure note to `[YOUR_LOG_PATH]\paper-digest-errors.log`:
```powershell
$errPath = "[YOUR_LOG_PATH]\paper-digest-errors.log"
"$(Get-Date -Format 'yyyy-MM-dd HH:mm') — Slack delivery failed. Digest saved to $outPath" | Add-Content -Path $errPath
```
3. Do NOT update the dedup log — papers will be retried on the next run.

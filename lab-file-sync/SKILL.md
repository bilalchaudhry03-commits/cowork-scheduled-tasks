---
name: lab-file-sync
description: Weekly file sync — upload staged files to cloud storage, scan for misplaced files, diff vs baseline, regenerate summary index, send Slack summary
---

> ⚠️ **This task is heavily lab-specific.** It was built around a particular folder schema and cloud storage setup. Use it as a reference pattern, not a drop-in solution. You will need to rewrite the paths, folder rules, and misplacement logic for your own setup. See the **Customization Guide** at the bottom.

Runs weekly to: (1) upload staged files to cloud storage, (2) detect misplaced files, (3) diff against last week's baseline, (4) regenerate a folder index, and (5) send a Slack completion ping.

## Setup — Customize These Values

| Placeholder | What to put here |
|---|---|
| `[YOUR_SLACK_CHANNEL_ID]` | Slack DM or channel ID to send the completion ping (e.g. `D0ADBJZCM0Q`) |
| `[YOUR_STAGING_PATH]` | Local folder where files are staged before upload (e.g. `C:\Users\you\staging`) |
| `[YOUR_CLOUD_PATH]` | Your cloud storage sync folder (OneDrive, Dropbox, etc.) |
| `[YOUR_INBOX_PATH]` | Destination subfolder in cloud storage for incoming files |
| `[YOUR_LOG_PATH]` | Local folder for scan logs and baseline file |
| `[YOUR_ARCHIVE_PATH]` | Local folder to move uploaded files after they've been sent |

## Required Tools
- Windows PowerShell — for file operations and cloud storage interaction
- Slack MCP connector (`send_message`) — for the completion ping

---

## STEP 0 — Upload staged files to cloud inbox

Scan `[YOUR_STAGING_PATH]` for top-level files (non-recursive). For each file found:
1. Copy it to `[YOUR_INBOX_PATH]`
2. Move the original to `[YOUR_ARCHIVE_PATH]`
3. Log the filename

```powershell
$src     = "[YOUR_STAGING_PATH]"
$inbox   = "[YOUR_INBOX_PATH]"
$archive = "[YOUR_ARCHIVE_PATH]"

New-Item -ItemType Directory -Force -Path $inbox   | Out-Null
New-Item -ItemType Directory -Force -Path $archive | Out-Null

$uploaded = [System.Collections.Generic.List[string]]::new()

Get-ChildItem -Path $src -File | ForEach-Object {
    Copy-Item -Path $_.FullName -Destination (Join-Path $inbox $_.Name) -Force
    Move-Item -Path $_.FullName -Destination (Join-Path $archive $_.Name) -Force
    $uploaded.Add($_.Name)
}

if ($uploaded.Count -eq 0) { Write-Host "No staged files." }
else { $uploaded | ForEach-Object { Write-Host "  Uploaded: $_" } }
```

---

## STEP 1 — Scan cloud storage for misplaced files

Write a full recursive file listing of `[YOUR_CLOUD_PATH]` to a temp file, then analyze it with Python to detect misplaced files.

```powershell
$root    = "[YOUR_CLOUD_PATH]"
$outFile = "[YOUR_LOG_PATH]\filelist_tmp.txt"

Get-ChildItem -Path $root -Recurse -File |
  Select-Object @{N='RelPath';E={$_.FullName.Substring($root.Length+1)}},
                @{N='Modified';E={$_.LastWriteTime.ToString('yyyy-MM-dd')}} |
  Sort-Object RelPath |
  ForEach-Object { "$($_.Modified)`t$($_.RelPath)" } |
  Out-File -FilePath $outFile -Encoding utf8
```

**Misplacement rules — adapt these to your folder conventions:**
- A `.py`, `.R`, `.ipynb` loose in a project root when a `scripts/` or `notebooks/` subfolder exists → misplaced
- A `.csv`, `.tsv`, `.xlsx` loose in a project root when `data/` or `results/` exists → misplaced
- A `.pptx` loose in a project root when `Presentations/` exists → misplaced
- A `.pdf` loose in a project root (not in a literature folder) → possibly misplaced

For each misplaced file: record current path, suggested destination, and one-line reason.

---

## STEP 2 — Diff against previous baseline

Read `[YOUR_LOG_PATH]\baseline.txt` (written by the previous run). Each line is a file path.

Compare against the current listing:
- **New files** — in current listing but not in baseline
- **Removed files** — in baseline but not in current listing

If `baseline.txt` does not exist, note "First run — no baseline available."

---

## STEP 3 — Regenerate folder index

Write `[YOUR_LOG_PATH]\YYYYMMDD_folder-index.md` as a markdown summary table — one row per top-level project folder:

| Folder | Files | Last Modified | Most Recent File |
|--------|-------|---------------|------------------|

Sort rows by Last Modified, newest first.

---

## STEP 4 — Write the scan log

Write `[YOUR_LOG_PATH]\scan-YYYY-MM-DD.md`:

- Files uploaded to cloud inbox: N (list filenames)
- New files since last run (bullet list or "None")
- Removed files since last run (bullet list or "None")
- Proposed moves (current path → suggested path + reason), or "No misplaced files found"
- Footer: "To execute any proposed moves, ask Claude directly in chat."

---

## STEP 5 — Update baseline

Overwrite `[YOUR_LOG_PATH]\baseline.txt` with a flat list of every file path currently in `[YOUR_CLOUD_PATH]` (one path per line, no dates or extra columns).

---

## STEP 6 — Send Slack completion ping

Send a Slack DM to `[YOUR_SLACK_CHANNEL_ID]`:

```
✅ *File sync complete — [YYYY-MM-DD]*
• Inbox upload: [N file(s) uploaded | No staged files]
• Misplaced files: [N found | None found]
• Changes since last run: [+N new] · [-N removed]
• Log: scan-[date].md
```

---

## Constraints
- Do NOT move, rename, or delete any files in cloud storage (Steps 1–5 read only; Step 0 copies to inbox only)
- Never overwrite a file already in cloud storage
- Do not ask clarifying questions — make reasonable judgments and note ambiguous cases in the log

---

## Customization Guide

This task was originally built for a specific lab setup. Here's what you'll most likely need to change:

**Folder schema** — The misplacement rules in Step 1 assume a particular project structure. Review and rewrite them to match how your lab or team organizes files.

**Cloud storage path** — Replace `[YOUR_CLOUD_PATH]` with wherever your cloud provider syncs locally (OneDrive, Dropbox, Google Drive, etc.).

**Staging folders** — This task assumes a single staging folder. If you have multiple (e.g. `active/` and `ready_for_review/`), extend the Step 0 loop to scan both.

**Slack channel** — Replace `[YOUR_SLACK_CHANNEL_ID]` with your own DM or channel ID. To find your DM channel ID: open Slack in a browser, click your own DM, and copy the channel ID from the URL (`/messages/D...`).

**Exclusions** — The original excluded raw data folders (`raw/`, `ref/`, `genome/`). Add or remove exclusion patterns in the PowerShell `Where-Object` filter to match your data conventions.

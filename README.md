# Cowork Scheduled Tasks

A collection of scheduled task prompts built for [Claude Cowork](https://claude.ai), Anthropic's desktop automation tool. Each task runs automatically on a schedule — no code required.

## What is a Cowork scheduled task?

In Claude Cowork, you can create tasks that run automatically (e.g. every morning at 9 AM, every Friday at 5 PM). Each task is defined by a prompt — a `SKILL.md` file — that tells Claude exactly what to do when it runs. Claude has access to connected tools like Slack, web search, and file operations.

Think of these as cron jobs, but written in plain English and executed by Claude.

## Tasks in this repo

### 🌍 [Daily World Briefing](./daily-world-briefing/SKILL.md)
**Schedule:** Every morning (e.g. 9 AM daily)

Searches the web for today's top world news, S&P 500 market data, and science/funding news, then sends a concise Slack DM you can read in under 2 minutes.

**Easiest to set up** — just swap in your Slack user ID and workspace name.

---

### 📄 [Daily Paper Digest](./daily-paper-digest/SKILL.md)
**Schedule:** Weekdays (e.g. 3 PM Mon–Fri)

Searches bioRxiv and PubMed across up to three research tracks you define, deduplicates against all previously seen papers, and sends a formatted Slack digest. Papers you've already seen never appear again.

**Best for:** researchers who want a daily literature feed on specific topics without sifting through noise.

To use: edit the **Search Topics** section with your own research keywords and journals.

---

### 🗂️ [Lab File Sync](./lab-file-sync/SKILL.md)
**Schedule:** Weekly (e.g. Fridays at 5 PM)

Uploads staged files to cloud storage, scans for misplaced files based on your folder conventions, diffs against last week's baseline, regenerates a folder index, and sends a Slack summary.

> ⚠️ This task is a reference pattern, not a plug-and-play template. It requires customization for your folder structure and cloud storage setup. Read the Customization Guide at the bottom of the SKILL.md before using.

---

## How to use these tasks

1. **Install Claude Cowork** — available at [claude.ai](https://claude.ai) (desktop app)
2. **Connect your tools** — link your Slack workspace and any other connectors the task needs (see each task's "Required Tools" section)
3. **Create a new scheduled task** in Cowork
4. **Paste the SKILL.md contents** as the task prompt, after filling in your `[CUSTOMIZE]` placeholders
5. **Set your schedule** — pick a time and recurrence (daily, weekdays, weekly, etc.)

That's it. Claude handles the rest automatically at each scheduled run.

## Customization

Each `SKILL.md` has a **Setup** table at the top listing every value you need to replace before running. Search for `[YOUR_` to find all placeholders.

## Requirements

- Claude Cowork (desktop app)
- Slack connected as an MCP tool (for tasks that send Slack messages)
- bioRxiv + PubMed MCP connectors (for the paper digest)
- Windows OS (PowerShell commands are used for local file operations)

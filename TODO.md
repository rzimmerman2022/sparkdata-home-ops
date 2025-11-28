# SparkData Home Ops - Master To-Do List

> **This is the SINGLE source of truth for all tasks.** All other docs (progress reports, handoffs, state files) reference back to this file.

---

## Completed Tasks

| Task | Model | Date | Time |
|------|-------|------|------|
| Set up Google Cloud service account (sda-cleaning-checklist) | Claude Opus 4.5 | 2025-11-26 | ~14:00 EST |
| Move credentials JSON to secure `.credentials/` folder | Claude Opus 4.5 | 2025-11-26 | ~14:10 EST |
| Create `.gitignore` to protect credentials and secrets | Claude Opus 4.5 | 2025-11-26 | ~14:15 EST |
| Create Google Apps Script for session logging | Claude Opus 4.5 | 2025-11-26 | ~14:20 EST |
| Fix Apps Script to use `openById()` instead of `getActiveSpreadsheet()` | Claude Opus 4.5 | 2025-11-28 | 00:30 UTC |
| Add Submit Session button with Google Sheets integration | Claude Opus 4.5 | 2025-11-26 | ~14:30 EST |
| Add notes textarea for cleaner comments | Claude Opus 4.5 | 2025-11-26 | ~14:30 EST |
| Add session summary modal with copy to clipboard | Claude Opus 4.5 | 2025-11-26 | ~14:30 EST |
| Test and confirm Google Sheets logging working | Claude Opus 4.5 | 2025-11-28 | 00:45 UTC |
| Merge DEV file changes to production | Claude Opus 4.5 | 2025-11-28 | 00:50 UTC |
| Push to GitHub main branch | Claude Opus 4.5 | 2025-11-28 | 00:50 UTC |
| Create handoff documentation (progress report, state file, handoff doc) | Claude Opus 4.5 | 2025-11-28 | 00:57 UTC |

---

## In-Progress Tasks

| Task | Status | Assigned |
|------|--------|----------|
| None | - | - |

---

## Pending Tasks

### High Priority
- None currently

### Medium Priority
- Add column headers to Google Sheet (Timestamp, Progress, Completed, Incomplete, Notes)
- Consider adding date picker for scheduling future cleaning sessions
- Add ability to export session history from Google Sheets to PDF

### Low Priority
- Add dark mode toggle for the checklist UI
- Consider adding photo upload capability for before/after documentation
- Add estimated time per task for better scheduling
- Add Apps Script email notifications when session is logged

---

## Project Resources

| Resource | Location |
|----------|----------|
| **Live Site** | GitHub Pages (sparkdata-home-ops) |
| **Google Sheet** | [SDA - Cleaning Session Logs](https://docs.google.com/spreadsheets/d/14sTCJCRBR1tQSadHzT3d1QSFoP8ZWasuJlRbGulldbM/edit) |
| **Apps Script URL** | `https://script.google.com/macros/s/AKfycbzqQXbXxDnOvo5MBMKAOVEFa_tOhemze0ma5C3LTM5DbwdpvUm0_SMnUymxmSPvP-4YBw/exec` |
| **Apps Script Owner** | ryan.zimmerman@southwestresumes.com |
| **Service Account** | sda-cleaning-checklist@srs-management-system.iam.gserviceaccount.com |

---

## Files

| File | Purpose |
|------|---------|
| `Deep_Clean_2025-11-26_Full_House_Single_Cleaner_Prioritized.html` | Production checklist |
| `Deep_Clean_2025-11-26_Full_House_Single_Cleaner_Prioritized_DEV.html` | Development/testing version |
| `.credentials/` | Secure storage for API keys (git-ignored) |
| `.gitignore` | Protects credentials from being committed |

---

## Handoff Documentation

| Document | Purpose |
|----------|---------|
| `PROGRESS_2025-11-28_00-57-30_opus-4.5_sheets-int-001.md` | Detailed progress report |
| `state/session_2025-11-28_00-57-30_opus-4.5_sheets-int-001.json` | Serialized session state for AI handoff |
| `docs/handoffs/HANDOFF_2025-11-28_00-57-30_opus-4.5_sheets-int-001.md` | Executive summary for next AI/developer |

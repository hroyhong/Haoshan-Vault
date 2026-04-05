# Morning Briefing — 9:00 AM

Copy this into Claude Code as a scheduled task.

**Folder:** Your vault root
**Repeats:** Every day at ~9:00
**Always allowed:** Bash

## Prompt

```
Check if a date file from yesterday exists at vault root (e.g. 2026-04-04.md). If it does, read it for any unfinished items, then delete the file. Create a new file at vault root named with today's date (e.g. 2026-04-05.md).

Contents of today's file:
1. Carried over — any unfinished items from yesterday's file
2. Calendar — check Google Calendar for today's events
3. Top priorities — read todo.md, pick the top 3 most urgent/important items
4. Forgotten — read journal.md latest entry, check if anything was mentioned but not in todo.md
5. Inbox classification — check Inbox/ root for unclassified items. Move each to _Important/, _Mid/, or _Remove/ based on content. If more than 5 unclassified items, prioritize the newest.

Keep the file concise. This is a daily dashboard, not a report.
```

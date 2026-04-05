# Daily Journal — 3:00 AM

Copy this into Claude Code as a scheduled task.

**Folder:** Your vault root
**Repeats:** Every day at ~3:00
**Always allowed:** Bash

## Prompt

```
Run git log --since="yesterday 3am" --until="now" --name-status and git diff HEAD@{1.day} on the vault repo. From the actual file changes, determine what was done yesterday: which files were created, modified, or deleted. Write a journal entry for yesterday summarizing the work. Move completed tasks in todo.md to 已完成 with yesterday's date. Check if any Life/Context cascade triggers were hit (new person mentioned, financial change, health event, relationship update, professional milestone) — if so, update the relevant files and their updated: frontmatter. Commit all changes with message "auto: close [yesterday's date]". No user interaction needed.
```

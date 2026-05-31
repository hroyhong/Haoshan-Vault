---
name: vault-init
description: Bootstrap an AI-orchestrated Obsidian vault with Chief of Staff persona, mode-specific project scaffold, scheduled routines, and persistent memory. Two modes ship: founder, researcher. Detects existing vaults and overlays non-destructively. Use when the user runs `/vault-init`, `/vault-init add-project <name>`, or `/vault-init add-character <name>`, or asks to set up a new AI operating system / personal vault / second brain.
---

# vault-init

You are running the `vault-init` skill. Your job is to bootstrap a personalized Obsidian vault wired with an AI Chief of Staff layer.

This is not a wiki. The system has three rules you will bake into the generated vault:
1. **Nothing enters the vault unprocessed.** Inbox/ is a queue with intent, not a dump.
2. **Notes/ is the user's thinking only.** AI never writes there.
3. **Every file has a goal or a goal-owner.** No orphans.

Lead the onboarding pitch with these rules. They are the product, not the feature list.

## Sub-commands

| Invocation | Behavior |
|---|---|
| `/vault-init` (no args) | Main bootstrap flow below |
| `/vault-init add-project <name>` | Add a Projects/<name>/ scaffold to existing vault, see "Incremental commands" |
| `/vault-init add-character <name>` | Add a CLAUDE.md persona to a folder, see "Incremental commands" |

If invoked with no args, run the main flow. If args present, jump to "Incremental commands" at the end.

## Main flow

### Step 1 — Pitch

Before any question, print this verbatim:

```
vault-init builds you an AI operating system for life + work.

This is not a wiki. Three rules:
  1. Nothing enters the vault unprocessed. Raw imports go to Inbox/,
     classified during your downtime. The Inbox is a queue, not a dump.
  2. Notes/ is your thinking only. The AI never writes there. Your
     thinking stays yours.
  3. Every file has a goal or a goal-owner. No orphan content.

What you get on top of those rules:
  - One AI that holds context across your projects, not 7 disconnected chats
  - Specialist characters (CTO, Thesis advisor) instead of cold-prompting GPT
  - Daily auto-briefing at 07:00, today's signal not noise
  - Persistent memory across sessions, your AI learns your preferences
  - Append-only journal, your future self gets the record automatically
  - Markdown on your machine, no SaaS lock-in, fully yours

Setup is tiered. 4 questions now, vault ships in 30 seconds.
Anything deferred is tracked and surfaced until done or declined.
```

### Step 2 — Tier 1 questions (essential)

Ask all four in one `AskUserQuestion` call. Required.

| Header | Question | Input shape |
|---|---|---|
| `Name` | What's your name (and how should I address you)? | freeform |
| `Mode` | Which mode fits you? | select: founder, researcher |
| `Vault path` | Where should the vault live? | freeform, default `~/Documents/<Name>Vault` |
| `Language` | Primary language for personas and reports? | select: en, zh, bilingual |

Treat the answers as `{{NAME}}`, `{{MODE}}`, `{{VAULT_PATH}}`, `{{LANG}}`. Derive `{{VAULT_NAME}}` from the last path component, `{{DATE}}` from today, `{{VAULT_SLUG}}` from the absolute path with `/` replaced by `-` (used for the MEMORY.md folder).

### Step 3 — Detect fresh vs integrate

Probe `{{VAULT_PATH}}`:
- Path does not exist OR exists and is empty → **fresh mode**
- Path contains `.obsidian/` directory OR any `*.md` file at top level → **integrate mode**

### Step 4a — Fresh mode

Create `{{VAULT_PATH}}` if needed. Walk `templates/common/` and `templates/{{MODE}}/`. For every file:
1. If filename ends in `.tmpl`, render by substituting `{{NAME}}` `{{MODE}}` `{{LANG}}` `{{VAULT_PATH}}` `{{VAULT_NAME}}` `{{DATE}}` and the Tier 2 placeholders (`{{STARTUP}}` for founder; `{{PROJECT}}` `{{LAB}}` `{{ADVISOR}}` for researcher; use literal `{{STARTUP}}` etc. if user defers Tier 2).
2. If folder name contains a placeholder, substitute it (or keep as literal if deferred).
3. Strip `.tmpl` suffix. Write to `{{VAULT_PATH}}/<relative path>`.
4. `.keep` files create the directory and are then deleted.
5. Inject the mode's `cos-persona.md` content into root `CLAUDE.md` where the `{{MODE_BLOCK}}` placeholder lives.

After file generation, proceed to Tier 2.

### Step 4b — Integrate mode

Show **Screen A — detected layout**: list top-level folders in `{{VAULT_PATH}}` and propose a mapping. Use defaults:
- `Projects/`, `Active/`, `Work/`, or similar → treat as the projects folder
- `Daily/`, `Daily Notes/`, `journal/` → keep, journal.md lives alongside
- `Notes/`, `notes/` → keep, do not write here
- `Inbox/`, `inbox/` → reuse, add `_Urgent/ _Read/ _Archive/ _morning/` subfolders if missing

Ask the user to confirm or override the mapping via `AskUserQuestion`.

Show **Screen B — write plan**: print every file you will create vs skip vs leave alone. Ask `Proceed?` via `AskUserQuestion` with options `Yes, proceed` and `No, abort`.

Iron rules:
- Never overwrite an existing file. Skip and tell.
- Never rename their folders. Adapt the generated `CLAUDE.md` to reference their names.
- Never `git init` if `.git/` exists. Print first-commit guidance instead.
- Cron name collisions: suffix `-2` and notify.

After confirmed write, proceed to Tier 2.

### Step 5 — Tier 2 questions (recommended, deferrable)

Ask in one `AskUserQuestion` batch. Each option must have a "Skip for now" alternative.

| Header | Question |
|---|---|
| `Main project` | Main project name? (Founder: startup name. Researcher: current research project / main paper line — not the dissertation, which gets added later.) Skip allowed. |
| `Biography` | Paste 2-3 lines of biography to seed `biography.md`? Skip allowed. |
| `Cadence` | Weekly review cadence? Options: Mon+Thu evenings, daily, weekly Sunday, skip. |

Apply answers in place (re-render relevant files) or write skipped items to `onboarding.md` (see Step 7).

### Step 6 — Tier 3 questions (optional, deferrable)

Ask in one `AskUserQuestion` batch. Bias toward skipping.

| Header | Question |
|---|---|
| `Sub-characters` | Add sub-characters to main project? (Founder: CTO, CPO, CMO. Researcher: just Thesis CoS by default.) multi-select. |
| `Life chars` | Add Life characters beyond Context? Options: Health, Therapist, Salon, Lab, Style. multi-select. |
| `Cron times` | Override default cron times? (07:00 briefing, 08:00 discovery, 22:30 journal) freeform or skip. |

### Step 7 — Write onboarding.md

For every Tier 2 / Tier 3 item that was skipped, append a `- [ ]` line under the right priority section in `{{VAULT_PATH}}/onboarding.md`. For every answered item, append `- [x] DONE {{DATE}}`. If the user declined an option, append `- [~] DECLINED {{DATE}}`.

The generated root `CLAUDE.md` already contains a block referencing `[[onboarding]]`. CoS will surface items session by session.

### Step 8 — Register cron routines

Use `CronCreate` three times. Schedule defaults (override from Tier 3 if user provided):

```
CronCreate({
  name: "{{VAULT_NAME}}-morning-briefing",
  schedule: "0 7 * * *",
  prompt: contents of templates/scheduled-tasks/morning-briefing/SKILL.md.tmpl rendered with {{VAULT_PATH}}
})
```

Repeat for `morning-discovery` (`0 8 * * *`) and `daily-journal` (`30 22 * * *`).

If `CronCreate` errors with name collision in integrate mode, retry with `-2` suffix.

### Step 9 — Seed MEMORY.md

Compute `{{VAULT_SLUG}}` = `{{VAULT_PATH}}` with `/` replaced by `-`. Target dir: `~/.claude/projects/{{VAULT_SLUG}}/memory/`.

If the dir does not exist, create it. Render and write:
- `MEMORY.md` (index)
- `user_profile.md` (name, language, mode)
- `user_mode.md` (mode-specific defaults)
- `vault_pointers.md` (paths to CoS, vault root, project folder)

Do NOT seed in integrate mode if the dir already exists with content — print a notice instead.

### Step 10 — git init + first commit

In fresh mode: `cd {{VAULT_PATH}} && git init && git add . && git commit -m "Initial vault from vault-init"`.
In integrate mode: skip if `.git/` exists, print guidance.

### Step 11 — Privacy notice + Obsidian wiring

Print:

```
Vault ready at {{VAULT_PATH}}

Privacy note:
  Your vault holds personal content (journal, profile, notes).
  Default .gitignore excludes raw personal files from git.
  If you publish this repo publicly, double-check .gitignore.
  To track everything (private repo only), remove entries from .gitignore.

Open in Obsidian:
  1. Launch Obsidian
  2. "Open folder as vault"
  3. Select {{VAULT_PATH}}

Routines scheduled:
  07:00 {{VAULT_NAME}}-morning-briefing
  08:00 {{VAULT_NAME}}-morning-discovery
  22:30 {{VAULT_NAME}}-daily-journal

To change: /schedule edit <name>
To disable: /schedule delete <name>

Next session: cd {{VAULT_PATH}} and start a fresh Claude Code conversation.
Your CoS persona is ready at root CLAUDE.md.

Open items in [[onboarding]] (N items). CoS will surface them session by session.
```

## Incremental commands

### `/vault-init add-project <name>`

1. Determine current vault root (cwd or look for `CLAUDE.md` going up).
2. Determine mode by reading root `CLAUDE.md` frontmatter (`mode: founder|researcher`).
3. Walk `templates/{{MODE}}/Projects/{{STARTUP_OR_PROJECT}}/` and render to `<vault>/Projects/<name>/` with `<name>` substituted for `{{STARTUP}}` (founder) or `{{PROJECT}}` (researcher).
4. Append `- [[<name>]]` to root `CLAUDE.md` Projects table.
5. If `onboarding.md` had `[ ] Main project name`, mark it `[x] DONE {{DATE}}`.

### `/vault-init add-character <character-name>`

1. Ask user where the character's CLAUDE.md should live (which folder).
2. Look up character template at `templates/characters/<character-name>.md.tmpl`. If not present, use a generic character template.
3. Write `<folder>/CLAUDE.md` and `<folder>/handoff.md`.
4. Update root `CLAUDE.md` character table.
5. If `onboarding.md` had a matching `[ ] Add <character>` line, mark it done.

## Template substitution

All `.tmpl` files use double-brace placeholders. Substitution is plain string replace.

| Placeholder | Source |
|---|---|
| `{{NAME}}` | Tier 1 answer |
| `{{MODE}}` | Tier 1 answer (founder or researcher) |
| `{{LANG}}` | Tier 1 answer (en, zh, bilingual) |
| `{{VAULT_PATH}}` | Tier 1 answer, absolute path |
| `{{VAULT_NAME}}` | Last component of VAULT_PATH |
| `{{VAULT_SLUG}}` | VAULT_PATH with `/` replaced by `-` |
| `{{DATE}}` | Today, YYYY-MM-DD |
| `{{STARTUP}}` | Tier 2 answer (founder mode) or kept literal if deferred |
| `{{PROJECT}}` | Tier 2 answer (researcher mode), main research project name |
| `{{LAB}}` | Tier 2 answer (researcher mode) or kept literal if deferred |
| `{{ADVISOR}}` | Tier 2 answer (researcher mode) or kept literal if deferred |
| `{{MODE_BLOCK}}` | Contents of `templates/{{MODE}}/cos-persona.md` |
| `{{BIOGRAPHY_SEED}}` | Tier 2 paste, or `(not yet provided)` if deferred |

Use literal substitution. Do not try to interpret markdown semantics.

## Language handling (v1)

v1 keeps templates English with placeholders. The user's language preference goes into root `CLAUDE.md` frontmatter so CoS responds in the chosen language in future sessions. Do NOT generate two copies of every file. Bilingual headers in section titles are fine where they appear in the source templates; do not invent new ones.

## What this skill never does

- Write to `Notes/` of the generated vault.
- Overwrite existing files in integrate mode.
- Force a specific folder name on the user.
- Auto-create a GitHub remote or auto-push.
- Touch the user's existing crons unless registering its own.

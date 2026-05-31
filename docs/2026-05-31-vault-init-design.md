---
status: design
date: 2026-05-31
author: Hong Haoshan + CoS
supersedes: static template repo (tagged static-template-v0)
---

# vault-init: a skill that bootstraps an AI-orchestrated personal operating system

## TL;DR

`vault-init` is a Claude Code skill. Run it once, answer four questions, get an Obsidian vault wired with a Chief of Staff persona, mode-specific project scaffold, scheduled daily routines, and persistent memory. Mode-specific personas ship for founders and researchers in v1. Anything you defer during onboarding gets tracked in `onboarding.md` and surfaced by CoS until done or declined.

The skill replaces the older static-template approach at `github.com/hroyhong/Haoshan-Vault` (preserved as tag `static-template-v0`).

## Why this exists

A vault full of markdown files is just storage. The leverage comes from the AI layer wrapped around it: a CoS persona that holds context across projects, character files that activate when you `cd` into a folder, daily crons that synthesize, and persistent memory that learns your preferences.

Setting that layer up by hand takes hours and depends on having a reference implementation to copy from. A static template repo solves part of this but freezes one person's choices into the file tree. The skill instead generates a vault personalized to the user's name, mode, language, and chosen sub-characters, with deferral and integration logic that a static repo cannot provide.

## Core values (what makes this different from a wiki)

**This is not a wiki.** A wiki dumps everything in one place and trusts search to surface what matters. The result is a graveyard of half-read pages and a search that returns ten near-duplicates. This system is the opposite shape.

Three rules define it.

1. **Nothing enters the vault unprocessed.** Raw imports, articles, screenshots, voice notes, all land in `Inbox/` and stay there until you classify them into `_Urgent/`, `_Read/`, `_Archive/`, or `Notes/`. The Inbox is a queue with intent, not a dump folder. CoS can help you classify but cannot promote on your behalf.
2. **Notes/ is your thinking only.** The AI never writes into `Notes/`. Not even as a draft for you to revise. If CoS has synthesis worth keeping, it goes to `Inbox/_Read/`; you write the `Notes/` entry from scratch in your own words. Your thinking stays yours. The system's job is to surface and synthesize, not to author your ideas.
3. **Every file has a goal or a goal-owner.** `Projects/` folders have a goal. `Life/` files describe a person whose state matters. There is no orphan content. Files without a goal get archived or deleted.

These three rules turn a folder tree into an operating system. Skip them and you have a wiki.

## What the user gets

When the user runs `/vault-init`, the skill prints a pitch:

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
```

Then four required questions, then the vault ships.

## Architecture

### Skill layout on disk

```
~/.claude/skills/vault-init/
  SKILL.md                       # The prompt Claude follows on /vault-init
  README.md                      # Install + usage instructions
  docs/
    2026-05-31-vault-init-design.md   # this file
  templates/
    common/                      # Shared across modes
      CLAUDE.md.tmpl             # Root CoS persona, {{MODE_BLOCK}} placeholder
      handoff.md.tmpl
      todo.md.tmpl
      journal.md.tmpl
      today.md.tmpl
      routines.md.tmpl
      README.md.tmpl
      onboarding.md.tmpl         # P2 + P3 deferred items
      Notes/.keep
      Inbox/
        _Urgent/.keep
        _Read/.keep
        _Archive/.keep
        _morning/.keep
      Life/
        Context/
          profile.md.tmpl
          biography.md.tmpl
    founder/
      Projects/
        {{STARTUP}}/
          {{STARTUP}}.md.tmpl     # Project description
          CLAUDE.md.tmpl          # Project CoS character
          plan.md.tmpl
          status.md.tmpl
          log.md.tmpl
          decisions-log.md.tmpl
          dashboard-feed.md.tmpl
          CTO/
            CLAUDE.md.tmpl        # AI character
            handoff.md.tmpl
          CPO/                    # human workspace, no CLAUDE.md
            users.md.tmpl
            plan.md.tmpl
          CMO/                    # human workspace, no CLAUDE.md
            outreach-emails.md.tmpl
          Outreach/.keep
        CFO/
          CFO.md.tmpl
          burn.md.tmpl
          runway.md.tmpl
        Customers/
          Customers.md.tmpl
        Hiring/
          Hiring.md.tmpl
        Conversations/
          Conversations.md.tmpl
        Knowledge/
          Knowledge.md.tmpl
      cos-persona.md              # Block injected into root CLAUDE.md {{MODE_BLOCK}}
    researcher/
      Projects/
        {{PROJECT}}/              # main research project; thesis added later via add-project
          {{PROJECT}}.md.tmpl
          CLAUDE.md.tmpl          # Project CoS character
          plan.md.tmpl
          status.md.tmpl
          log.md.tmpl
          code/README.md          # analysis scripts, model code, notebooks
          data/README.md          # raw + processed; large data pointed to outside vault
          literature/README.md    # papers cited in this project
          drafts/README.md        # paper drafts, organized by target venue
          figures/README.md       # output plots, schematics
        {{LAB}}/
          {{LAB}}.md.tmpl
        {{ADVISOR}}/
          {{ADVISOR}}.md.tmpl
        Papers/
          Papers.md.tmpl          # cross-project pipeline view
          CLAUDE.md.tmpl          # Papers pipeline character
        Conferences/
          Conferences.md.tmpl
        Grants/
          Grants.md.tmpl
        Teaching/
          Teaching.md.tmpl
        Collaborators/
          Collaborators.md.tmpl
        Conversations/
          Conversations.md.tmpl
        Knowledge/
          Knowledge.md.tmpl
      cos-persona.md
    scheduled-tasks/
      morning-briefing/SKILL.md.tmpl
      morning-discovery/SKILL.md.tmpl
      daily-journal/SKILL.md.tmpl
    memory/
      MEMORY.md.tmpl              # Seed for ~/.claude/projects/<vault-slug>/memory/
      user_profile.md.tmpl
      user_mode.md.tmpl
```

### Project vs character distinction

Two markers define a folder's role.

| Markers present | Folder is |
|---|---|
| `<X>/<X>.md` only | Pure project, no AI persona (e.g. `Conversations/`) |
| `<X>/CLAUDE.md` only | Pure character, sub-domain of parent (e.g. `Fluentide/CTO/`) |
| Both | Orchestrated project (e.g. `Fluentide/Fluentide.md` + `Fluentide/CLAUDE.md`) |
| Neither | Not a project, just storage (e.g. `Notes/`, `Inbox/`) |

The skill bakes this rule into the generated tree. Sub-folders are deliberately split: `CTO/` gets `CLAUDE.md`, `CPO/` does not.

## Onboarding flow

### Tier 1 — Essential (blocks shipping)

The skill batches all four questions into one `AskUserQuestion` call.

| Q | Input | Notes |
|---|---|---|
| Name | freeform | Used in CoS persona greetings, MEMORY.md seed |
| Mode | select: founder, researcher | Determines which `templates/<mode>/` is copied |
| Vault path | freeform | Default `~/Documents/<Name>Vault`, validated |
| Language | select: en, zh, bilingual | Affects all template substitutions |

### Tier 2 — Recommended (deferrable, tracked)

After Tier 1, skill writes the skeleton vault, then asks three more in one screen with a Skip option per item.

| Q | Input | If deferred |
|---|---|---|
| Main project name (Startup / research project) | freeform or skip | `{{STARTUP}}` / `{{PROJECT}}` placeholder remains; tracked |
| Seed biography from CV | paste text or skip | `biography.md` stays stub; tracked |
| Weekly cadence (Mon/Thu reviews? daily?) | select or skip | `routines.md` stays stub; tracked |

### Tier 3 — Optional (deferrable, tracked)

Asked last with bias toward skipping.

| Q | Input | If deferred |
|---|---|---|
| Add CTO/CPO/CMO sub-characters? | multi-select or skip-all | Per-character add tracked individually |
| Add Life characters beyond Context? | multi-select: Health, Therapist, Salon, Lab, Style | Per-character add tracked individually |
| Custom cron times | overrides or skip | Defaults stay (07:00, 08:00, 22:30); tracked as optional revisit |

### Fresh vs integrate detection

Before writing any file, the skill probes the chosen vault path.

```
empty or nonexistent  → fresh mode, full stack
contains .obsidian/   → integrate mode, overlay
contains any *.md     → integrate mode, overlay
```

Integrate mode adds two confirmation screens before any write.

**Screen A — detected layout.** Skill scans top-level folders, prints a mapping table, and asks for overrides.

```
Detected: Active/, Daily/, References/, Inbox/

Plan:
  Active/      → treated as Projects/ (CLAUDE.md will reference Active/)
  Daily/       → kept as-is; journal.md will live alongside
  References/  → kept as-is, not touched
  Inbox/       → reused for Inbox/_Urgent/ etc., subfolders added

Override any mapping? [y/N]
```

**Screen B — write plan.** Skill prints exactly which files it will create vs skip vs leave alone.

```
Will create:
  ./CLAUDE.md       (root CoS persona, references your folder names)
  ./handoff.md
  ./today.md
  ./onboarding.md
  ./Life/Context/profile.md
  ./Inbox/_Urgent/ _Read/ _Archive/ _morning/
  ~/.claude/scheduled-tasks/<vault>-morning-briefing/SKILL.md

Will skip (already exists):
  ./journal.md     → your file kept; CLAUDE.md will reference it
  ./todo.md        → your file kept

Will NOT touch:
  ./Active/*       → your existing work untouched
  ./References/*

Proceed? [y/N]
```

### Iron rules for integrate mode

- Never overwrite an existing file. Skip and tell.
- Never rename their folders. Adapt the generated CLAUDE.md to reference their names.
- Never `git init` if `.git/` exists. Print first-commit guidance.
- Never silently overwrite a cron. If a name collides, suffix with `-2` and notify.

## Deferral and the onboarding file

Anything deferred from Tier 2 or Tier 3 lands in `onboarding.md` at vault root.

```markdown
# Onboarding (in progress)

## P2 — Recommended
- [ ] Main project name (using placeholder {{STARTUP}} for now)
- [ ] Seed biography.md from CV
- [ ] Decide weekly review cadence

## P3 — Optional
- [ ] Add CTO sub-character to main project
- [ ] Add Health character
- [ ] Custom cron times
- [x] DONE 2026-05-31  Set language to bilingual
- [~] DECLINED 2026-05-31  Add Style character
```

The root CLAUDE.md gets a block that makes CoS treat this file as a session-start checklist.

```markdown
### Onboarding in progress
Open items live in [[onboarding]]. At session start, check the file:
  - P1 pending → block: don't proceed until resolved
  - P2 pending → mention at session start, propose tackling one
  - P3 pending → mention only at end-of-session check-in
Mark [x] DONE <date> when file evidence shows completion.
Mark [~] DECLINED <date> when user says skip.
When all items resolved, move onboarding.md to Inbox/_Archive/ and delete this block.
```

This single mechanism also handles "add later" decisions. Three weeks after a deferred CTO add, CoS still surfaces it. The user can then run `/vault-init add-character CTO` as an incremental command.

## Routines

The skill ships three cron routines, all on by default, all editable.

| Routine | Default | Output |
|---|---|---|
| `<vault>-morning-briefing` | 07:00 daily | Writes `today.md` |
| `<vault>-morning-discovery` | 08:00 daily | Writes `Inbox/_morning/<date> <title>.md` |
| `<vault>-daily-journal` | 22:30 daily | Appends to `journal.md` |

Each routine has two artifacts:

1. `~/.claude/scheduled-tasks/<vault>-<task>/SKILL.md` — the prompt the cron fires
2. Cron registration via `CronCreate({ name, schedule, prompt })`

Mode does not change cron prompts in v1. The SKILL.md prompt references the vault's CLAUDE.md persona, which already encodes mode-specific focus. So a founder vault's briefing surfaces sprint and burn because its CoS persona is sprint and burn aware. Per-mode cron logic is a v2 idea.

## Memory.md collaboration

Claude Code auto-creates one memory folder per working directory, keyed by absolute path:

```
~/.claude/projects/-Users-<user>-Documents-<VaultName>/memory/MEMORY.md
```

Slug rule: take the absolute path, replace `/` with `-`. The vault at `/Users/alex/Documents/AlexVault` maps to `~/.claude/projects/-Users-alex-Documents-AlexVault/memory/`.

`MEMORY.md` is an index, not memory itself. Each remembered fact is its own file in the same folder, indexed as `- [Title](file.md) — hook`.

The skill seeds three files at install:

```
MEMORY.md               # index, one-line header + 3 links
user_profile.md         # name, address-as, language, mode
user_mode.md            # mode-specific defaults (sprint cadence vs grant cadence)
vault_pointers.md       # paths to CoS, mode, vault root
```

Without seeding, Claude Code creates the folder on first save anyway. Seeding preloads first-session context so the user does not have to teach the AI their name or mode after install.

## Distribution and install

The skill is distributed as a git repo.

```bash
git clone https://github.com/hroyhong/Haoshan-Vault ~/.claude/skills/vault-init
```

After install, `/vault-init` becomes available in Claude Code. The repo replaces the older static template (preserved as tag `static-template-v0`).

To update: `cd ~/.claude/skills/vault-init && git pull`. Updates affect future `/vault-init` runs but never modify existing vaults. The skill writes once and the user owns the output.

## Incremental commands

v1 ships three sub-commands beyond the main flow.

| Command | Purpose |
|---|---|
| `/vault-init` | Main bootstrap (fresh or integrate) |
| `/vault-init add-project <name>` | Add a Projects/<name>/ scaffold inside existing vault |
| `/vault-init add-character <name>` | Add a CLAUDE.md persona to an existing folder |

`add-mode` is deferred to v2. Users who need a non-shipped mode edit their own root CLAUDE.md.

## Privacy and gitignore

The skill writes a sensible `.gitignore` at vault root:

```
# Personal content
journal.md
todo.md
Notes/
Inbox/_Urgent/
Inbox/_Read/
Inbox/_morning/
Inbox/_Archive/
Life/Context/profile.md
Life/Context/biography.md

# Obsidian
.obsidian/workspace*
.obsidian/cache
.trash/

# OS
.DS_Store
```

Tracked: vault structure, CLAUDE.md files, project scaffolds. Untracked: raw personal content.

At end of onboarding, the skill prints:

```
Privacy note:
  Your vault holds personal content (journal, profile, notes).
  Default .gitignore excludes raw personal files from git.
  If you publish this repo publicly, double-check .gitignore.
  To track everything (private repo only), remove entries from .gitignore.
```

The skill does not auto-push or auto-create a remote. The user controls when and where.

## What v1 deliberately does not ship

- `add-mode` incremental command (edit root CLAUDE.md manually)
- Per-mode cron prompts (mode flavor comes from CoS persona)
- Mode authoring via `templates/custom/` (v2)
- Generic mode (founder + researcher only; revisit if users ask)
- Auto-Obsidian symlink (skill prints instructions, user runs)
- Vault dashboard (`dashboard.html` family from the source vault, not ported in v1)
- Health/Therapist/Salon/Lab/Style CLAUDE.md files (available as `add-character` after install)

## Acceptance criteria

The v1 skill is done when:

1. Running `/vault-init` in an empty directory produces a working vault with personalized CoS persona, generated Projects/ scaffold, seeded MEMORY.md, registered crons, first git commit.
2. Running `/vault-init` in a directory with `.obsidian/` enters integrate mode, prints the two confirmation screens, and respects all skip-existing-files rules.
3. Deferring all Tier 2 + Tier 3 items results in a complete `onboarding.md` and a CoS block in root CLAUDE.md that surfaces those items.
4. Running `/vault-init add-character CTO` inside the main project folder creates `<project>/CTO/CLAUDE.md` and updates root CLAUDE.md character table.
5. `.gitignore` is written and the privacy notice prints at end of onboarding.
6. The skill repo at `github.com/hroyhong/Haoshan-Vault` install instructions work end-to-end from a fresh machine.
7. **The three core values are visible in three places:** repo README (so visitors see them before installing), the onboarding pitch (so the user reads them before the first question), and the generated root CLAUDE.md (so CoS enforces them every session). The generated root CLAUDE.md contains explicit rules: `Notes/` is human-only, `Inbox/` requires explicit classification before promotion, no orphan content.

## Open questions for implementation

- Template syntax: `{{NAME}}` mustache-style is simplest but conflicts with literal `{{` in markdown. Alternative: `<<NAME>>` or `$NAME`. Mustache-style chosen for v1, escape with `\{{` if needed.
- Should the skill detect existing `~/.claude/skills/vault-init` and refuse double-install or auto-update? v1 refuses with message.
- Bilingual mode: do we generate two copies of every persona file, or one file with both languages inline? v1 inline (matches Haoshan's existing vault style).

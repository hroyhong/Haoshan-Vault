# haoshan-vault

Bootstrap an AI-orchestrated Obsidian vault with a Chief of Staff persona, mode-specific project scaffold, daily scheduled routines, and persistent memory. One command, four questions, ready in thirty seconds.

## This is not a wiki

Three rules define the system. They are the product.

1. **Nothing enters the vault unprocessed.** Raw imports go to `Inbox/`, classified during your downtime. The Inbox is a queue with intent, not a dump folder. The AI can help you classify but cannot promote on your behalf.
2. **`Notes/` is your thinking only.** The AI never writes there. Not even as a draft for you to revise. If the AI has synthesis worth keeping, it goes to `Inbox/_Read/`; you write the Notes/ entry yourself, in your own words. Your thinking stays yours.
3. **Every file has a goal or a goal-owner.** `Projects/` folders have goals. `Life/` files describe a person whose state matters. There is no orphan content.

If you want a wiki, search engines exist. This is an operating system.

## What you get

- **Chief of Staff persona** at vault root. Holds context across all your projects, surfaces conflicts, preps decisions.
- **Specialist characters** that activate when you `cd` into a folder: CTO for code review, Thesis CoS for chapter work, Papers for submission discipline.
- **Three daily cron routines**: morning briefing (07:00), discovery synthesis (08:00), evening journal (22:30). All editable.
- **Persistent memory** at `~/.claude/projects/<vault-slug>/memory/`. Your AI learns your preferences, past decisions, failure modes across sessions.
- **Two modes ship in v1**: `founder` (startup, CTO/CPO/CMO, CFO, runway) and `researcher` (thesis, lab, advisor, papers, conferences, grants).
- **Fresh or integrate**: empty directory gets the full stack; existing vault gets a non-destructive overlay.
- **Markdown on your machine**, no SaaS lock-in, fully yours.

## Install

Open Claude Code and paste this in plain English:

> Install the haoshan-vault skill from github.com/hroyhong/Haoshan-Vault into ~/.claude/skills/haoshan-vault, then run /haoshan-vault.

Claude reads the request, clones the repo, registers the skill, and starts the four-question onboarding. You don't type a single shell command yourself — the AI handles it.

To update later, paste:

> Update the haoshan-vault skill.

Claude pulls the latest version. Updates affect future `/haoshan-vault` runs only. Existing vaults are never modified.

## Usage

```
/haoshan-vault                              # main bootstrap (fresh or integrate)
/haoshan-vault add-project <name>           # add a Projects/<name>/ to existing vault
/haoshan-vault add-character <character>    # add a CLAUDE.md persona to a folder
```

## Onboarding flow

Four required questions, then the vault ships:

1. **Name** (and how to address you)
2. **Mode**: founder or researcher
3. **Vault path** (default `~/Documents/<Name>Vault`)
4. **Language**: en / zh / bilingual

After the vault is generated, two optional question rounds. Anything you skip lands in `onboarding.md` and gets surfaced by your CoS in future sessions until you finish it or decline.

## What the generated vault looks like

```
{{VaultName}}/
  CLAUDE.md                 # CoS persona, knows your mode and name
  handoff.md
  todo.md
  journal.md
  today.md                  # auto-written by morning-briefing cron
  routines.md
  onboarding.md             # open items from Tier 2 + Tier 3
  Notes/                    # your thinking only, AI never writes here
  Inbox/
    _Urgent/                # decisions and proposals needing your reply
    _Read/                  # AI synthesis for you to read
    _Archive/               # marked for deletion
    _morning/               # daily discovery output
  Projects/
    <mode-specific scaffold>
  Life/
    Context/
      profile.md            # stable facts about you
      biography.md          # history
```

The mode-specific Projects/ scaffold depends on what you picked.

**Founder mode** ships: `<Startup>/` with `CTO/`, `CPO/`, `CMO/`, `Outreach/`; plus `CFO/`, `Customers/`, `Hiring/`, `Conversations/`, `Knowledge/`.

**Researcher mode** ships: `<Project>/` with the standard research-project shape (`code/`, `data/`, `literature/`, `drafts/`, `figures/`); plus `<Lab>/`, `<Advisor>/`, `Papers/` (cross-project pipeline), `Conferences/`, `Grants/`, `Teaching/`, `Collaborators/`, `Conversations/`, `Knowledge/`. The dissertation gets added later via `/haoshan-vault add-project Thesis` once chapters start synthesizing across projects.

## Crons

Three routines register automatically. You can edit times in Tier 3 or with `/schedule edit <name>` later.

| Routine | Default time | Output |
|---|---|---|
| morning-briefing | 07:00 | Writes `today.md` |
| morning-discovery | 08:00 | Writes one synthesis note to `Inbox/_morning/` |
| daily-journal | 22:30 | Appends to `journal.md` |

## Privacy

The skill writes a default `.gitignore` that excludes raw personal content (`journal.md`, `todo.md`, `Notes/`, `Inbox/`, `Life/Context/profile.md`, `Life/Context/biography.md`) from git. Vault structure and CLAUDE.md files are tracked.

If you publish the vault repo publicly, double-check `.gitignore` before pushing. The skill never auto-creates a remote or auto-pushes.

## Older static template

A previous static-template version of this repo is preserved at tag `static-template-v0`:

```bash
git checkout static-template-v0
```

It was a fixed file tree without onboarding. The skill replaces it.

## Author

Hong Haoshan, building Fluentide while training to be a Chief of Staff for himself.

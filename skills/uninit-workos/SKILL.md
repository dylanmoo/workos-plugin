---
name: uninit-workos
description: "Reverse what /init-workos did, removing only the scaffold files it created and leaving every other document and folder untouched. Use whenever the user says 'uninit workos,' 'undo init-workos,' 'remove the workos scaffold,' 'tear down workos,' 'reset this back,' 'revert init-workos,' or asks to clean up the CLAUDE.md / MEMORY.md / ARCHIVE.md scaffolding without losing their existing content. Identifies scaffold files by a sentinel marker (`workos-init-scaffold v1`) that init-workos writes at the top of every file it creates. Files without the sentinel — including any document the user wrote — are never deleted or modified. When a scaffold file has been edited since init, shows the user what changed and asks before removing."
---

# Uninit WorkOS

The reverse of `/init-workos`. Walks the workspace, removes only the scaffold files that `/init-workos` created, and leaves every other file and folder untouched.

## What this skill does

Three things, and only three things:

1. **Finds** every scaffold file by grepping for the sentinel `workos-init-scaffold v1` that `/init-workos` writes at the top of each file it creates.
2. **Classifies** each scaffold file as clean (unchanged from the original template) or edited (the user added content after init).
3. **Removes** clean scaffold files automatically; asks the user file-by-file what to do with edited ones.

Anything without the sentinel — pre-existing documents, files the user wrote later, files created by `/create-project` — is never touched.

## The sentinel

`/init-workos` writes this exact line at the top of every file it creates (after YAML frontmatter when present):

```html
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
```

This skill matches the substring `workos-init-scaffold v1`. Anything containing that token is a scaffold file. Anything not containing it is user content and stays.

If the user removed the sentinel manually, the file looks like user content to this skill — it will be left alone. That's intentional: when in doubt, preserve.

## Step 1: Confirm the workspace root

Use the current working directory as the workspace root. State the path back to the user and confirm before scanning.

## Step 2: Find all scaffold files

Recursively grep the workspace for files containing `workos-init-scaffold v1`. Limit to depth 3 (root → domain → project, or root → domain → group → project) to match what `/init-workos` could have created. Filenames to inspect:

- `CLAUDE.md`
- `MEMORY.md`
- `ARCHIVE.md`

Skip the same exclusion lists as init-workos: `.git`, `.github`, `.obsidian`, `.claude`, `.vscode`, `.idea`, `node_modules`, `dist`, `build`, `target`, `__pycache__`, `.venv`, `venv`, `plugins`, `skills`.

If zero scaffold files are found, stop and tell the user: "No `/init-workos` scaffold detected here. Nothing to undo."

## Step 3: Classify each scaffold file

For each file with the sentinel, classify it:

- **Clean** — the file matches the original init-workos template exactly (modulo the date in the sentinel). Safe to delete with no questions.
- **Edited** — the file contains content beyond what init-workos wrote. The user added something after init.

A rough heuristic for "clean":
- Body length within 10% of the canonical template length for that file type
- No headings or bullet items outside the canonical template
- Placeholder lines (`*(nothing active yet)*`, `*(none yet)*`, `*(empty)*`) still present

If you can't confidently classify, default to **Edited** (safer — forces a user decision rather than silent deletion).

## Step 4: Show the user the plan

Present the findings as a single block:

```
Found 9 scaffold files from /init-workos:

  Clean — safe to remove (5):
    ./CLAUDE.md
    ./ARCHIVE.md
    ./personal/CLAUDE.md
    ./personal/MEMORY.md
    ./clients/CLAUDE.md

  Edited — needs your decision (4):
    ./MEMORY.md                       (added: "Active Projects" entries, 12 lines)
    ./clients/MEMORY.md               (added: 2 entries under Key Decisions)
    ./clients/acme-corp/CLAUDE.md     (added: Workflow steps)
    ./clients/acme-corp/MEMORY.md     (added: Contacts section content)

Proceed? Three options:
  (a) Remove all clean files; ask file-by-file about edited ones
  (b) Remove only the clean files; leave edited ones alone
  (c) Cancel
```

## Step 5: Handle clean files

For each file in the clean list, delete it. Report the count when done.

No folder cleanup — `/init-workos` doesn't create folders (only files), so there's nothing to remove on the folder side. If the user wants to delete now-empty folders, that's a separate explicit ask.

## Step 6: Handle edited files (one at a time)

For each edited file, show:

1. The file path
2. A brief diff: which sections / lines are additions beyond the template

Then ask: "What do you want to do with this file?"

- **Delete it** — remove the whole file, including the user's additions. (Confirm once more before doing this.)
- **Keep it, strip the scaffold** — remove the sentinel line and any unmodified template sections, preserving the user's additions. Useful when the user wants to keep what they wrote but stop using WorkOS.
- **Leave it alone** — no changes. The file stays exactly as-is, sentinel and all.

When stripping, be conservative: only remove sections that are unchanged from the template. If a section has been edited even slightly, keep it.

## Step 7: Confirm and report

Show the user a summary:

```
Removed:
  ./CLAUDE.md
  ./ARCHIVE.md
  ./personal/CLAUDE.md
  ./personal/MEMORY.md
  ./clients/CLAUDE.md
  ./clients/acme-corp/MEMORY.md   (kept user content; stripped scaffold sections)

Left alone:
  ./MEMORY.md                     (user chose: leave alone)
  ./clients/MEMORY.md             (user chose: leave alone)
  ./clients/acme-corp/CLAUDE.md   (user chose: leave alone)

No other files or folders were touched.
```

End with one line: "Workspace is back to its pre-init state for the files you removed. You can re-run `/init-workos` any time."

## Optional: backup before removing

If the user wants a safety net, offer once at the start: "Want me to copy the scaffold files into `.workos-uninit-backup/` before removing them, in case you change your mind?"

If yes, mirror the directory structure inside `.workos-uninit-backup/<timestamp>/` and copy each file before deletion. If no, proceed without backup.

## What this skill does NOT do

- **Doesn't delete files without the sentinel.** Anything the user wrote, anything `/create-project` created, anything pre-existing — all left alone.
- **Doesn't delete folders.** `/init-workos` doesn't create folders, so there's nothing to remove on the folder side. Empty subfolders the user wants to clean up are a separate manual step.
- **Doesn't touch `raw/` or `resources/` contents.** `/init-workos` never wrote there.
- **Doesn't uninstall the WorkOS plugin or any other skills.** Those are managed at the Claude Desktop level, not by this skill.
- **Doesn't ask the user to identify scaffold files manually.** The sentinel is authoritative; if a file doesn't have it, it's not a scaffold file.
- **Doesn't reverse anything besides `/init-workos`.** It's not a general workspace teardown — it's specifically the inverse of init.

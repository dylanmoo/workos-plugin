---
name: init-workos
description: "Initialize a WorkOS-style workspace on top of an existing directory that already has files organized by project. Use whenever the user says 'init workos,' 'set up workos here,' 'initialize my workspace,' 'bootstrap workos,' 'scaffold workos,' 'convert this to workos,' or asks to lay the WorkOS structure over a folder they already use. Recursively scans the existing directory to detect domains (1 level deep), groups (2 levels deep — folders that bundle related workstations), and workstations (2 or 3 levels deep depending on whether they sit directly under a domain or inside a group). Creates root CLAUDE.md (with routing map), MEMORY.md, ARCHIVE.md, plus per-domain CLAUDE.md + MEMORY.md, plus per-group CLAUDE.md + MEMORY.md when groups exist, plus per-workstation CLAUDE.md + MEMORY.md for every project. Never moves or modifies existing files without explicit approval. Narrates each step so the user understands what's being created and why. One-time bootstrap; for ongoing maintenance use create-workstation, session-audit, or workspace-audit."
---

# Init WorkOS

A one-time bootstrap skill that lays a WorkOS scaffold over an existing project-organized directory. Run once per workspace; afterward use `/create-workstation`, `/session-audit`, and `/workspace-audit` for ongoing maintenance.

## Narration — explain as you go

This skill writes files into the user's workspace. **Tell the user what you're doing and why at every step.** Each Step below has a "Narrate:" line — that's the message to send before performing the step. Don't be silent between actions. Treat the user as a smart collaborator who wants to understand the moves you're making with their files.

Default tone: brief, plain-spoken, one or two sentences per checkpoint. State the action and the reason. Example: "Scanning your folders to classify each one — domains, groups, workstations, or content storage. Won't write anything yet." Not example: "Initiating recursive directory traversal at depth 3."

## What this skill does

1. **Recursively detects** the existing folder shape — which top-level folders are domains, which subfolders are groups that bundle related workstations, which folders are workstations (individual projects), and which are content storage.
2. **Creates** files at up to four levels: root, domain, group (when present), and workstation.
3. **Wires the routing maps** at every level (root → domains, domain → groups + workstations, group → workstations) so future sessions auto-route to the right context.

Existing files are never moved, renamed, or deleted. The skill only adds new files.

## The four levels

| Level | Path | Files written | Routes to... |
|---|---|---|---|
| Root | `<workspace>/` | `CLAUDE.md`, `MEMORY.md`, `ARCHIVE.md` | Domains |
| Domain | `<workspace>/<domain>/` | `CLAUDE.md`, `MEMORY.md` | Groups and/or workstations |
| Group (optional) | `<workspace>/<domain>/<group>/` | `CLAUDE.md`, `MEMORY.md` | Workstations inside the group |
| Workstation | `<workspace>/<domain>/<workstation>/` OR `<workspace>/<domain>/<group>/<workstation>/` | `CLAUDE.md`, `MEMORY.md` | (leaf — no further routing) |

A workspace might be two levels (root → domain), three (root → domain → workstation), or four (root → domain → group → workstation). Different domains can mix shapes freely — one domain can have grouped workstations while another has direct workstations or none at all.

## Sentinel marker (used by every scaffold file)

Every file this skill creates begins with a single-line HTML comment sentinel — placed as the very first line of the file, except when the file has YAML frontmatter (then the sentinel goes immediately after the closing `---`). The sentinel lets `/uninit-workos` safely identify which files came from this skill so it can clean up without touching anything else.

Format (substitute today's date):

```html
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
```

Add this line at the top of each of the templates below (after frontmatter where present). Do not invent variants; the exact `workos-init-scaffold v1` token is what `/uninit-workos` matches against.

## Step 1: Confirm the workspace root

**Narrate:** "I'm going to scaffold a WorkOS workspace over `<cwd>`. That means adding a few markdown files that make Claude smarter session-over-session by giving it persistent memory and routing. I won't move or modify any of your existing files, and the whole scaffold is reversible via `/uninit-workos`. OK to scan your folder structure?"

Use the current working directory as the workspace root. State the path back to the user and wait for confirmation before scanning.

If a root `CLAUDE.md` already exists with a "Routing Map" section, the workspace is already a WorkOS — stop and tell the user. Suggest `/workspace-audit` for health-checking or `/create-workstation` for adding a new domain.

## Step 2: Recursive scan — detect domains, groups, and workstations

**Narrate:** "Scanning your folders to classify each one. I'm looking at three things: domains (top-level areas of work), groups (folders that bundle related workstations together — like `clients/` containing multiple client folders), and workstations (individual projects or engagements). I'm also flagging content folders like `raw/` and `resources/` as storage, not scaffolding targets. No files are being written yet."

Walk the directory tree to three levels below the root. For each folder, classify it.

### Skip lists (never scaffolded — not a domain, group, or workstation)

- Hidden folders (start with `.`): `.git`, `.github`, `.obsidian`, `.claude`, `.vscode`, `.idea`
- Build / dependency folders: `node_modules`, `dist`, `build`, `target`, `__pycache__`, `.venv`, `venv`
- WorkOS infrastructure at root: `plugins`, `skills`

### Content-folder names (never scaffolded — they're storage inside a domain or group)

`raw`, `resources`, `private`, `briefs`, `notes`, `attachments`, `images`, `docs`, `archive`, `src`, `lib`

### Classification rules

- **Top-level folder under root** → domain candidate (unless on the skip list above).
- **Subfolder inside a domain candidate** (level 2):
  - Has a content-folder name → content, skip.
  - Contains files (any file type, not counting `.DS_Store`, `.gitkeep`, or a lone `README.md`) → workstation candidate.
  - Contains only subfolders (no meaningful direct files) AND at least one of those subfolders looks like a workstation → **group candidate**. Descend into it.
  - Empty → ambiguous, flag for the user.
- **Subfolder inside a group candidate** (level 3):
  - Has a content-folder name → content, skip.
  - Contains files → workstation candidate.
  - Anything else → ambiguous, flag for the user.
- **Anything deeper than level 3** → not scaffolded automatically. The user can run `/create-workstation` for those after init completes.

### When in doubt, ask

Don't silently assume. If a folder is ambiguous (empty, mixed signals, name doesn't fit a clear category), surface it to the user during Step 3 and let them decide.

## Step 3: Show the tree and confirm

**Narrate:** "Here's what I found. Look it over and tell me if I got anything wrong — what's a group vs a workstation, whether anything I'm planning to skip should be scaffolded, or anything that should be added that I missed."

Present the detected tree in a single block. Annotate each entry with what it is and why:

```
Detected structure (proposed scaffold):

  personal/                              — domain, 3 loose files
  work/                                  — domain (empty so far)
  oneday/                                — domain
    ├── clients/                         — group, 3 workstations inside
    │   ├── acme-corp/                   — workstation (12 files)
    │   ├── widgetco/                    — workstation (8 files)
    │   └── crossgate/                   — workstation (5 files)
    ├── workshops/                       — group, 2 workstations inside
    │   ├── becoming-superhuman/         — workstation (3 files)
    │   └── zero-to-live/                — workstation (4 files)
    ├── resources/                       — content folder (skipped)
    └── raw/                             — content folder (skipped)
  research/                              — domain
    ├── papers/                          — content folder (skipped)
    └── 2026-launch-research/            — workstation (5 files)

Skipping: .git/, node_modules/, .obsidian/, plugins/, skills/

Confirm this structure? You can:
  - Remove anything from the scaffold (it stays on disk; just doesn't get a CLAUDE.md)
  - Promote a "content folder" to a workstation or group
  - Reclassify a "workstation" as a group, or vice versa
  - Add a new domain, group, or workstation that doesn't exist yet
```

Resolve any user adjustments before writing.

## Step 4: Gather routing conditions

**Narrate:** "For each domain, group, and workstation, I need a one-line description of when future sessions should load it. This is what powers the routing maps — when you start a session and say what you're working on, Claude reads these descriptions and loads the right folder's context automatically."

For each confirmed **domain**, ask:

> "When should future sessions auto-route to `<domain>/`? Phrase as '...working on X' or '...need to do Y.'"

For each confirmed **group**, ask:

> "When should future sessions route to the `<domain>/<group>/` group? This catches the case where the user is working on something inside the group but hasn't named a specific workstation yet."

For each confirmed **workstation**, ask:

> "When should future sessions route to the `<full-path>/` workstation?"

Propose a default derived from the folder name when the user can't articulate one. Examples:

- `clients/` → "...working on any client engagement"
- `clients/acme-corp/` → "...working on the Acme Corp engagement"
- `workshops/` → "...preparing or refining any workshop"
- `research/2026-launch-research/` → "...researching the 2026 launch"

Keep the asks lean. Don't gather identity paragraphs or workflow steps here — those refine later via `/create-workstation` or hand-edits.

## Step 5: Write root files

**Narrate:** "Writing three files at the workspace root: `CLAUDE.md` (rules + routing map to your domains), `MEMORY.md` (current state, read at every session start), and `ARCHIVE.md` (historical log, only read when you ask). Every file gets a sentinel comment so `/uninit-workos` can cleanly remove this scaffold later if you change your mind."

Create three files at the workspace root. **If any already exists, stop and ask before overwriting.**

### `CLAUDE.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# CLAUDE.md — [Workspace Name, defaults to folder basename]

A WorkOS-style workspace. Rules live here, current state in `MEMORY.md`, historical record in `ARCHIVE.md`, durable per-domain knowledge in `{domain}/resources/`, source material in `{domain}/raw/`.

## Memory System

At the start of every session, read `MEMORY.md` before responding. Use what you find to inform your work — don't announce what you found, just act on it.

When the user says "remember this," write the information to `MEMORY.md` immediately and confirm.

**Where things go.**

1. **Does it prescribe behavior?** ("always," "never," "before doing X, do Y") → `CLAUDE.md` (root, domain, group, or workstation — most-specific scope wins).
2. **Does it describe a mutable fact?** (contacts, status, decisions) → `MEMORY.md` (root, domain, group, or workstation).
3. **Is it durable knowledge** true on a six-month horizon? → the right file in `{domain}/resources/`.

**Memory hygiene rules.**

1. Keep each memory entry to two sentences max.
2. Keep root `MEMORY.md` under 150 lines; when breached, compress first, then archive.
3. When something completes or becomes outdated, move from `MEMORY.md` to `ARCHIVE.md`.
4. `ARCHIVE.md` is reference-only: never read at session start, only when the user asks about something historical.

## Rules

### Memory cascade

Domain-scoped memory goes to that domain's `MEMORY.md`. Group-scoped memory goes to the group's `MEMORY.md`. Workstation-scoped memory goes to that workstation's `MEMORY.md`. Root `MEMORY.md` is for cross-domain context only.

### Rules MECE

Rules across `CLAUDE.md` files must be mutually exclusive (no rule appears in two places) and collectively exhaustive (the rule set together covers every situation that warrants a rule). When adding a rule, place it once in the most-specific applicable file.

## Routing Map

When starting a task, check this table to determine which domain folder to load.

| Domain | Route here when the user... |
|---|---|
| `<domain1>/` | [routing condition from Step 4] |
| `<domain2>/` | [routing condition from Step 4] |
| ... |

Each domain has its own `CLAUDE.md` and `MEMORY.md`. Domains with nested groups or workstations have their own routing map inside their `CLAUDE.md`. Cross-domain synthesis only on explicit request.

## References

| Resource | Read when... |
|---|---|
| `MEMORY.md` | Always — every session, before responding |
| `{domain}/MEMORY.md` | Working in that domain |
| `{domain}/<group>/MEMORY.md` | Working in that group |
| `{domain}/<workstation>/MEMORY.md` or `{domain}/<group>/<workstation>/MEMORY.md` | Working in that workstation |
| `{domain}/resources/*.md` | Working in that domain — durable knowledge |
| `{domain}/raw/*` | Need source material; grep before asserting |
```

### `MEMORY.md`

```markdown
---
name: WorkOS MEMORY
description: Root memory file. Mutable state about the user, their roles, and active themes. Read first by every agent session.
scope: root
---
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->

# WorkOS Memory

Last updated: [today's date in YYYY-MM-DD]

Two-sentence-max entries. 150-line ceiling — when breached, compress, then move historical entries to `ARCHIVE.md`.

## Active Projects

*(nothing active yet — add entries as work begins)*

## Core Memory

*(things the agent has learned about the user over time — say "remember this" during any session to add an entry)*
```

### `ARCHIVE.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# WorkOS Archive

Historical record. Reference-only — never read at session start, only when the user asks about something historical.

*(empty)*
```

## Step 6: Write per-domain files

**Narrate (once, before the loop):** "Now writing CLAUDE.md and MEMORY.md for each domain. If a domain has groups or workstations inside it, the domain's CLAUDE.md gets its own routing map for them."

For each confirmed domain, create `<domain>/CLAUDE.md` and `<domain>/MEMORY.md`. **If either already exists, stop and ask before overwriting.**

### `<domain>/CLAUDE.md`

The template has two parts: a base block (always written), and an inline routing map (only when the domain contains groups or workstations).

**Base template:**

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# CLAUDE.md — <domain> domain

[One-sentence description, derived from folder name. If ambiguous, ask the user for a one-liner.]

## What's here

- Existing files at this domain's root (preserved as-is from before WorkOS init).
- `resources/` — distilled themes and durable knowledge (create on first use).
- `raw/` — source material (create on first use).
[If the domain has groups, append:]
- Group subfolders (each bundles related workstations). Each group has its own `CLAUDE.md` with rules that apply to all workstations inside it, plus a routing map.
[If the domain has direct workstations (not inside a group), append:]
- Workstation subfolders (one per project or engagement). Each has its own `CLAUDE.md` and `MEMORY.md`.
[Use both lines if the domain has both groups and direct workstations.]

## How to work here

- Default context: `<domain>` only.
- Filenames may contain spaces or special chars — quote paths.

## Memory cascade

Domain-wide facts → this folder's `MEMORY.md`. Group-wide facts → that group's `MEMORY.md`. Workstation-specific facts → that workstation's `MEMORY.md`. Don't write more-specific state at a less-specific level.
```

**Additional Routing Map section, only when the domain has groups or workstations:**

```markdown
## Routing Map

When starting a task in this domain, check this table to load the right group or workstation.

| Path | Route here when... |
|---|---|
| `<group1>/` | [group routing condition from Step 4] |
| `<group1>/<workstation>/` | [workstation routing condition from Step 4] |
| `<direct-workstation>/` | [workstation routing condition from Step 4] |
| ... |
```

The table mixes groups and workstations in a single column. Show group rows above their nested workstation rows for readability. Use the full relative path from the domain (e.g., `clients/acme-corp/` not just `acme-corp/`) so the path is unambiguous from any cwd inside the domain.

### `<domain>/MEMORY.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# <domain> Memory

Mutable state for the `<domain>` domain. Two-sentence-max entries. When something completes or ages out, move to root `ARCHIVE.md`.

## Active work

*(nothing active yet)*

## Key decisions

*(none yet)*
```

## Step 7: Write per-group files

**Narrate (once, before the loop):** "Now writing CLAUDE.md and MEMORY.md for each group. A group is a non-workstation folder that bundles related workstations — like `clients/` containing multiple client folders. The group CLAUDE.md is where you put rules that apply to ALL workstations inside it (e.g., 'every client engagement starts with a discovery call'). Each group also gets its own routing map for the workstations inside."

Skip this step entirely if no groups were detected.

For each confirmed group, create `<domain>/<group>/CLAUDE.md` and `<domain>/<group>/MEMORY.md`. **If either already exists, stop and ask before overwriting.**

### `<domain>/<group>/CLAUDE.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# CLAUDE.md — <group> group (in <domain>)

[One-sentence description of what this group bundles. If ambiguous, ask the user.]

## What's here

A group of related workstations under the `<domain>` domain. Each workstation has its own `CLAUDE.md` and `MEMORY.md`. This file holds rules and context shared across every workstation in the group.

## How to work here

- Default context: this group + its parent domain.
- Group-wide rules go here. Workstation-specific rules go in each workstation's `CLAUDE.md`.
- When a workstation contradicts the group, the workstation wins (most-specific scope).

## Routing Map

| Workstation | Route here when... |
|---|---|
| `<workstation1>/` | [routing condition from Step 4] |
| `<workstation2>/` | [routing condition from Step 4] |
| ... |
```

### `<domain>/<group>/MEMORY.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# <group> Memory

Mutable state shared across workstations in the `<group>` group. Two-sentence-max entries. Workstation-specific state belongs in that workstation's `MEMORY.md`, not here.

## Active work

*(nothing active yet)*

## Key decisions

*(none yet)*
```

## Step 8: Write per-workstation files

**Narrate (once, before the loop):** "Last batch — writing CLAUDE.md and MEMORY.md for each workstation. The workstation CLAUDE.md has placeholders for Identity, Resources, Workflow, and Editorial Rules. The Workflow placeholder is TBD on purpose — fill it in the first time you do real work in that workstation, so it reflects what actually happened, not a guess."

For each confirmed workstation, create CLAUDE.md and MEMORY.md at its full path (`<domain>/<workstation>/` for direct workstations, `<domain>/<group>/<workstation>/` for grouped). **If either already exists, stop and ask before overwriting.**

### `<workstation-path>/CLAUDE.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# CLAUDE.md — <workstation>

## Identity

You are the user's partner for the `<workstation>` [engagement / project / area]. Future sessions route here when [routing condition from Step 4].

## Resources

| Resource | Read when... |
|---|---|
| *(none yet)* | *(populate as durable reference files arrive)* |

## Workflow

1. *TBD — refine on first use.*

## Editorial Rules

Follow voice principles in the workspace root (and any `personal/resources/voice.md` if present). Layer workstation-specific writing rules here as they emerge.
```

### `<workstation-path>/MEMORY.md`

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# <workstation> Memory

Mutable state for the `<workstation>` [engagement / project]. Two-sentence-max entries. When something completes, move to the domain's `ARCHIVE.md` (or root `ARCHIVE.md` if the workstation is closed out entirely).

## Active work

*(nothing active yet)*

## Contacts

*(none yet)*

## Key decisions

*(none yet)*
```

## Step 9: Confirm and hand off

**Narrate:** "Done. Here's everything I created and where each routing map points. A few quick next steps — and a reminder you can always run `/uninit-workos` if you want to back any of this out."

Show the user:

1. The full list of files created, grouped by level (root / domain / group / workstation), with paths.
2. The routing-map rows added to root `CLAUDE.md`, each domain `CLAUDE.md`, and each group `CLAUDE.md`.
3. Three next steps:
   - "Open the root `CLAUDE.md` and skim it. Adjust anything that doesn't fit your style."
   - "Skim each domain, group, and workstation `CLAUDE.md` — fill in the Workflow step in each workstation the first time you do real work there."
   - "Start any future session by saying what you're working on — the routing maps will load the right context. Run `/create-workstation` to add a new workstation later. Run `/uninit-workos` if you want to remove this scaffold."

If anything was ambiguous during creation, surface it and ask the user to clarify before declaring done.

## What this skill does NOT do

- **Doesn't move or rename existing files.** Loose files stay where they are. Migration into `raw/` is a separate, manual decision.
- **Doesn't create empty `raw/` or `resources/` folders.** Those are created on first use, not pre-emptively.
- **Doesn't scaffold deeper than workstation level (3 below root).** If a workstation has its own internal subfolders, those are left alone — they're the workstation's internal organization, not separate workstations.
- **Doesn't infer rules, workflows, or identity paragraphs.** The user adds those as they emerge.
- **Doesn't overwrite existing `CLAUDE.md`, `MEMORY.md`, or `ARCHIVE.md` files.** Always asks first.
- **Doesn't install plugins or other skills.** Assumes the WorkOS plugin is already installed.
- **Doesn't re-run on a workspace that already has a root `CLAUDE.md` with a Routing Map.** Stops and routes the user to `/workspace-audit` or `/create-workstation`.
- **Doesn't enforce IP/privacy boundaries between domains.** If you need to keep one domain's context out of another, add that rule manually to the root `CLAUDE.md` after init.

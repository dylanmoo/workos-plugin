---
name: init-workos
description: "Initialize a WorkOS-style workspace on top of an existing directory that already has files organized by project. Use whenever the user says 'init workos,' 'set up workos here,' 'initialize my workspace,' 'bootstrap workos,' 'scaffold workos,' 'convert this to workos,' or asks to lay the WorkOS structure over a folder they already use. Recursively scans the existing directory to detect domains (1 level deep) and nested workstations (2 levels deep). Creates root CLAUDE.md (with routing map), MEMORY.md, ARCHIVE.md, plus per-domain CLAUDE.md + MEMORY.md, plus per-workstation CLAUDE.md + MEMORY.md when nested project folders exist. Never moves or modifies existing files without explicit approval. One-time bootstrap; for ongoing maintenance use create-workstation, session-audit, or workspace-audit."
---

# Init WorkOS

A one-time bootstrap skill that lays a WorkOS scaffold over an existing project-organized directory. Run once per workspace; afterward use `/create-workstation`, `/session-audit`, and `/workspace-audit` for ongoing maintenance.

## What this skill does

Three things, and only three things:

1. **Recursively detects** the existing folder shape — which top-level folders should become domains, and within each domain, which nested folders should become workstations.
2. **Creates** files at three possible levels:
   - **Root**: `CLAUDE.md`, `MEMORY.md`, `ARCHIVE.md`
   - **Domain**: `CLAUDE.md`, `MEMORY.md` (plus a domain-level routing map if workstations live inside)
   - **Workstation**: `CLAUDE.md`, `MEMORY.md`
3. **Wires the routing maps** at both root and domain levels so future sessions auto-route to the right context.

Existing files are never moved, renamed, or deleted. The skill only adds new files.

## The three levels

| Level | Path | Files written | Routes to... |
|---|---|---|---|
| Root | `<workspace>/` | `CLAUDE.md`, `MEMORY.md`, `ARCHIVE.md` | Domains |
| Domain | `<workspace>/<domain>/` | `CLAUDE.md`, `MEMORY.md` | Workstations inside it (if any) |
| Workstation | `<workspace>/<domain>/<workstation>/` | `CLAUDE.md`, `MEMORY.md` | (leaf — no further routing) |

A workspace might have two levels (root → domain) or three (root → domain → workstation), or a mix where some domains have workstations and others don't.

## Step 1: Confirm the workspace root

Use the current working directory as the workspace root. State the path back to the user and confirm before writing anything.

If a root `CLAUDE.md` already exists with a "Routing Map" section, the workspace is already a WorkOS — stop and tell the user. Suggest `/workspace-audit` for health-checking or `/create-workstation` for adding a new domain.

## Step 2: Recursive scan — detect domains and workstations

Walk the directory tree to depth 2. For each folder, classify it:

### Skip lists (never count as a domain or workstation)

- Hidden folders (start with `.`): `.git`, `.github`, `.obsidian`, `.claude`, `.vscode`, `.idea`
- Build / dependency folders: `node_modules`, `dist`, `build`, `target`, `__pycache__`, `.venv`, `venv`
- WorkOS infrastructure at root: `plugins`, `skills`

### Content-folder names (never count as workstations — they're storage inside a domain)

`raw`, `resources`, `private`, `briefs`, `notes`, `attachments`, `images`, `docs`, `archive`, `src`, `lib`

If a folder inside a domain has one of these names, it's content for the domain, not a nested workstation.

### Classification rules

- **Top-level folder under root** → domain candidate (unless on the skip list).
- **Subfolder inside a domain candidate**:
  - Has a content-folder name → content, skip.
  - Contains files (any file type, not just markdown) → workstation candidate.
  - Empty, or contains only further nested folders → ambiguous, flag for the user.
- **Anything deeper than 2 levels** → don't scaffold. If the user wants a workstation that deep, they run `/create-workstation` manually later.

For each domain candidate, also note: does it itself have any files at its root (not in subfolders)? If yes, those are "loose" domain-level files that stay as-is.

## Step 3: Show the tree and confirm

Present the detected tree in a single block. Annotate each entry with what was found:

```
Detected structure (proposed scaffold):

  personal/                       — domain, 3 loose files
  work/                           — domain (empty so far)
  clients/                        — domain
    ├── acme-corp/                — workstation candidate (12 files)
    ├── widgetco/                 — workstation candidate (8 files)
    └── notes/                    — content folder (skipped)
  research/                       — domain
    ├── papers/                   — content folder (skipped)
    └── 2026-launch-research/     — workstation candidate (5 files)
  mana/                           — domain
    └── (no nested workstations)

Skipping: .git/, node_modules/, .obsidian/, plugins/, skills/

Confirm this structure? You can:
  - Remove any domain or workstation from the scaffold
  - Promote a folder from "content" to "workstation"
  - Add a new domain or workstation name
```

Resolve any user adjustments before writing. If a workstation candidate is ambiguous (empty folder, mixed signals), explicitly ask before treating it as a workstation.

## Step 4: Gather routing conditions

For each confirmed **domain**, ask:

> "When should future sessions auto-route to `<domain>/`? Phrase as '...working on X' or '...need to do Y.'"

For each confirmed **workstation**, ask:

> "When should future sessions route to `<domain>/<workstation>/`?"

If the user can't articulate, propose one from the folder name and confirm. Examples:

- `clients/` → "...working on any client engagement"
- `clients/acme-corp/` → "...working on the Acme Corp engagement"
- `research/` → "...doing research, reading, or note-taking"
- `research/2026-launch-research/` → "...researching the 2026 launch"

Keep the asks lean. Don't gather identity paragraphs or workflow steps here — those refine later via `/create-workstation` or hand-edits.

## Step 5: Ask about IP/privacy boundaries

One question, only at the domain level (workstations inherit their domain's boundaries):

> "Should any of these domains be isolated from each other — meaning Claude shouldn't pull context from one when working in another?"

If yes, collect the pairs (e.g., "`work/` never enters `side-project/`"). These become hard rules in the root `CLAUDE.md`.

If no, note it inline and move on.

## Sentinel marker (used by every scaffold file)

Every file this skill creates begins with a single-line HTML comment sentinel — placed as the very first line of the file, except when the file has YAML frontmatter (then the sentinel goes immediately after the closing `---`). The sentinel lets `/uninit-workos` safely identify which files came from this skill so it can clean up without touching anything else.

Format (substitute today's date):

```html
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
```

Add this line at the top of each of the templates below (after frontmatter where present). Do not invent variants; the exact `workos-init-scaffold v1` token is what `/uninit-workos` matches against.

## Step 6: Write root files

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

1. **Does it prescribe behavior?** ("always," "never," "before doing X, do Y") → `CLAUDE.md` (root, domain, or workstation — most-specific scope wins).
2. **Does it describe a mutable fact?** (contacts, status, decisions) → `MEMORY.md` (root, domain, or workstation).
3. **Is it durable knowledge** true on a six-month horizon? → the right file in `{domain}/resources/`.

**Memory hygiene rules.**

1. Keep each memory entry to two sentences max.
2. Keep root `MEMORY.md` under 150 lines; when breached, compress first, then archive.
3. When something completes or becomes outdated, move from `MEMORY.md` to `ARCHIVE.md`.
4. `ARCHIVE.md` is reference-only: never read at session start, only when the user asks about something historical.

## Rules

### Memory cascade

Domain-scoped memory goes to that domain's `MEMORY.md`, not root. Workstation-scoped memory goes to that workstation's `MEMORY.md`, not the domain's. Root `MEMORY.md` is for cross-domain context only.

### Rules MECE

Rules across `CLAUDE.md` files must be mutually exclusive (no rule appears in two places) and collectively exhaustive (the rule set together covers every situation that warrants a rule). When adding a rule, place it once in the most-specific applicable file.

[If Step 5 produced isolation pairs, include the following. Otherwise omit this section.]

### Domain boundaries (IP/privacy firewall)

When operating in one domain, do NOT pull context from another unless the user explicitly asks for a cross-domain synthesis.

- [`<domain-A>` content NEVER enters `<domain-B>`. Reason: <user's stated reason>.]
- [additional pairs as collected]

Enforcement is the agent's responsibility. CWD signals the active domain; honor it.

## Routing Map

When starting a task, check this table to determine which domain folder to load.

| Domain | Route here when the user... |
|---|---|
| `<domain1>/` | [routing condition from Step 4] |
| `<domain2>/` | [routing condition from Step 4] |
| ... |

Each domain has its own `CLAUDE.md` and `MEMORY.md`. Domains with nested workstations have their own routing map inside their `CLAUDE.md`. Cross-domain synthesis only on explicit request.

## References

| Resource | Read when... |
|---|---|
| `MEMORY.md` | Always — every session, before responding |
| `{domain}/MEMORY.md` | Working in that domain |
| `{domain}/<workstation>/MEMORY.md` | Working in that workstation |
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

## Step 7: Write per-domain files

For each confirmed domain, create `<domain>/CLAUDE.md` and `<domain>/MEMORY.md`. **If either already exists, stop and ask before overwriting.**

### `<domain>/CLAUDE.md`

The template branches based on whether the domain has nested workstations.

**Base template (all domains):**

```markdown
<!-- workos-init-scaffold v1 — created by /init-workos on YYYY-MM-DD. Remove via /uninit-workos. -->
# CLAUDE.md — <domain> domain

[One-sentence description, derived from folder name. If ambiguous, ask the user for a one-liner.]

## What's here

- Existing files at this domain's root (preserved as-is from before WorkOS init).
- `resources/` — distilled themes and durable knowledge (create on first use).
- `raw/` — source material (create on first use).
[If the domain has nested workstations, append:]
- Workstation subfolders (one per project or engagement). Each has its own `CLAUDE.md` and `MEMORY.md`. The routing map below points future sessions to the right one.

## How to work here

- Default context: `<domain>` only. [If Step 5 added an isolation pair involving this domain, append: "Do not pull from `<other-domain>/`."]
- Filenames may contain spaces or special chars — quote paths.

## Memory cascade

Domain-wide facts → this folder's `MEMORY.md`. Workstation-specific facts → that workstation's `MEMORY.md`. Don't write workstation-specific state at the domain level.
```

**Additional section, only if the domain has workstations:**

```markdown
## Routing Map

When starting a task in this domain, check this table to load the right workstation.

| Workstation | Route here when... |
|---|---|
| `<workstation1>/` | [routing condition from Step 4] |
| `<workstation2>/` | [routing condition from Step 4] |
| ... |
```

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

## Step 8: Write per-workstation files

For each confirmed workstation, create `<domain>/<workstation>/CLAUDE.md` and `<domain>/<workstation>/MEMORY.md`. **If either already exists, stop and ask before overwriting.**

### `<domain>/<workstation>/CLAUDE.md`

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

Follow voice principles in the workspace root (`~/<workspace-root>/MEMORY.md` and any `personal/resources/voice.md` if present). Layer workstation-specific writing rules here as they emerge.
```

### `<domain>/<workstation>/MEMORY.md`

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

Show the user:

1. The full list of files created, grouped by level (root / domain / workstation), with paths.
2. The routing-map rows added to root `CLAUDE.md` and each domain `CLAUDE.md`.
3. Three next steps:
   - "Open the root `CLAUDE.md` and skim it. Adjust anything that doesn't fit your style."
   - "Skim each domain and workstation `CLAUDE.md` — fill in the workflow step when you know your first one."
   - "Start any future session by saying what you're working on — the routing maps will load the right context. Run `/create-workstation` to add a new workstation later, or re-run `/init-workos` only if the root scaffold doesn't exist yet."

If anything was ambiguous during creation, surface it and ask the user to clarify before declaring done.

## What this skill does NOT do

- **Doesn't move or rename existing files.** Loose files stay where they are. Migration into `raw/` is a separate, manual decision.
- **Doesn't create empty `raw/` or `resources/` folders.** Those are created on first use, not pre-emptively.
- **Doesn't scaffold deeper than workstation level (2 below root).** Deeper nesting is rare; the user can run `/create-workstation` manually if they need it.
- **Doesn't infer rules, workflows, or identity paragraphs.** The user adds those as they emerge.
- **Doesn't overwrite existing `CLAUDE.md`, `MEMORY.md`, or `ARCHIVE.md` files.** Always asks first.
- **Doesn't install plugins or other skills.** Assumes the WorkOS plugin is already installed.
- **Doesn't re-run on a workspace that already has a root `CLAUDE.md` with a Routing Map.** Stops and routes the user to `/workspace-audit` or `/create-workstation`.

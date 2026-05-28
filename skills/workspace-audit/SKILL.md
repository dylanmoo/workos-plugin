---
name: workspace-audit
description: "Audits a WorkOS workspace for drift — misplaced entries, size-ceiling violations, archive candidates, format violations, and engagement-specific content that should cascade to a workstation. Use when the user says 'workspace audit,' 'audit my workspace,' 'check my CLAUDE.md,' 'check my MEMORY.md,' 'is my workspace healthy,' or when periodically maintaining a WorkOS-style workspace. Distinct from session-audit, which only captures new learnings from the current conversation — this skill audits the workspace files themselves."
---

# Starter Workspace Audit

A periodic health check for any WorkOS-style workspace. Catches drift before it costs token usage or output quality.

## What This Skill Does

Five checks, then a single grouped report. Never writes changes without explicit approval.

1. **Wrong-location check.** Factual entries in `CLAUDE.md` (should move to `MEMORY.md`) and prescriptive entries in `MEMORY.md` (should move to `CLAUDE.md`).
2. **Archive candidacy check.** `MEMORY.md` entries that look historical or stale and should move to `ARCHIVE.md`.
3. **Size ceiling check.** Current line counts vs. target (`CLAUDE.md` target 200–250, max 300; `MEMORY.md` max 150).
4. **Entry format check.** `MEMORY.md` entries longer than two sentences.
5. **Cascade check.** Engagement-, project-, or domain-specific entries in root `MEMORY.md` that should live in a deeper-level `MEMORY.md`.

## Step 1: Discover the Workspace Root

Find the WorkOS root dynamically. Look for a `CLAUDE.md` with a Routing Map section in the mounted workspace. That's the root.

If the user named a specific workstation to audit (e.g., "audit my Email HQ workstation"), audit that one instead of the root.

If you can't find a root, stop and tell the user — don't invent structure.

## Step 2: Read the Files

For the chosen scope, read:

- `CLAUDE.md`
- `MEMORY.md`
- `ARCHIVE.md` (if it exists)

If `ARCHIVE.md` doesn't exist at the root level, flag that as the first finding before running any other check — the WorkOS isn't fully set up and several other checks depend on its existence.

## Step 3: Run the Five Checks

### A. Wrong-location check

For each entry in `CLAUDE.md`:

- If its primary purpose is recording a fact or status (a name, a date, a project status, "this person works at X"), flag it for move to `MEMORY.md`.

For each entry in `MEMORY.md`:

- If its primary purpose is prescribing behavior ("always," "never," "before doing X, do Y," "when X happens, do Y"), flag it for move to `CLAUDE.md`.

### B. Archive candidacy check

For each entry in `MEMORY.md`:

- Does it describe a completed project, an outdated decision, or context that's been superseded?
- Is it about a contact or event that's no longer active?

Check the Memory hygiene / Memory System section in `CLAUDE.md` for the workspace's specific rules on what stays vs. archives. Different WorkOS instances may define "current-state content" differently — don't impose a generic rule.

**Skip entries that match a "stays regardless of age" rule** (often: active projects, contacts, working conventions, identity facts).

### C. Size ceiling check

Report current line counts as a table:

- `CLAUDE.md`: actual / target 200–250 / max 300
- `MEMORY.md`: actual / max 150
- `ARCHIVE.md`: actual (no ceiling)

If either is over target, note it but don't propose specific cuts here — that's what the other checks are for. Once the user applies findings from checks A, B, D, and E, ceilings often come back under target naturally.

### D. Entry format check

For each bullet or entry in `MEMORY.md`:

- Count sentences. Flag any entry over two.
- Propose a compressed version that captures the same fact in one or two sentences.

### E. Cascade check

For each entry in root `MEMORY.md`:

- Does the entry describe a specific workstation, engagement, or sub-domain (not generic root-level context)?
- If yes, does that workstation have its own `MEMORY.md`?
- If the workstation `MEMORY.md` exists, propose moving the detail there and replacing the root entry with a short pointer (e.g., "Latest status in `[Workstation Name]/MEMORY.md`").
- If no workstation `MEMORY.md` exists yet, flag it as a candidate for `create-workstation`.

## Step 4: Present Findings

Group findings by check. Use this format:

```
**Workspace Audit Report — [date] — [scope: root / workstation name]**

## A. Wrong-location entries

[N]. **[Entry summary]**
- Currently in: `[file]`
- Should be in: `[file]`
- Proposed wording: [exact text]
- Why: [one sentence]

## B. Archive candidates

[same format]

## C. Size ceiling status

| File | Lines | Target | Status |
|---|---|---|---|
| CLAUDE.md | [X] | 200–250 (max 300) | ✓ / ⚠ over target / ❌ over max |
| MEMORY.md | [X] | max 150 | ✓ / ❌ over max |
| ARCHIVE.md | [X] | n/a | n/a |

## D. Entry format violations

[same format]

## E. Cascade opportunities

[same format]
```

If a check returns no findings, say so under that section header. Don't omit the section.

If there are no findings overall, report "Clean workspace. Nothing to move." Don't manufacture work.

## Step 5: Apply Approved Changes

After the user approves (all, some, or none of the findings), write the approved changes. For each change, confirm what was written and where:

- Moves between files (cut from source, paste to destination)
- Compressed entries (replace verbose version with compressed)
- Archive transfers (cut from MEMORY.md, paste to ARCHIVE.md under the right section)
- Cascade transfers (cut from root MEMORY.md, paste to workstation MEMORY.md, leave a one-line pointer in root)

**Important:** Never write changes without approval. Always present findings first and wait.

## When to Run This Skill

- **Weekly** during the first month of a new WorkOS setup — drift is most likely while patterns are still forming
- **Monthly** after that
- **Before a major migration** (e.g., when adopting new patterns from a video, course, or article)
- **When output quality slips** — often a symptom of CLAUDE.md or MEMORY.md drift, not the model

## What This Skill Does NOT Do

- Doesn't audit per-workstation files unless explicitly asked. Default scope is the root.
- Doesn't merge or split workstations. If a workstation looks misaligned with how it's actually being used, surface it as a finding but don't try to fix it programmatically.
- Doesn't change the schema or section structure of any file. It moves entries; it doesn't redesign.
- Doesn't run other skills as part of its flow. If a cascade finding suggests creating a new workstation, surface that as a recommendation and let the user invoke `create-workstation` separately.

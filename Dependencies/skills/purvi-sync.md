---
name: purvi-sync
description: Use when the user says "product sync", "start a sync", "let's sync", "purvi sync", or similar. Runs a dedicated product sync conversation — the user drives discussion, the AI captures patterns, principles, and decisions. Ends when the user says "Assembly dispersed."
---

# Purvi Product Sync

A dedicated product sync session. The user drives the conversation. You capture patterns, principles, and decisions. Nothing gets written without approval.

**Purvi repo path:** Read from `/root/.claude/ground-truths.md` if it exists, otherwise default to `/root/Purvi/`.

**If Purvi is not found:** Tell the user "Purvi is not installed on this machine. Can't run a product sync." Stop.

## Starting the sync

### 1 — Pull latest and load context

```bash
cd <purvi-path> && git pull --ff-only 2>&1
```

If pull fails, report the error. Continue with local state but warn the user that the remote may have newer content.

Read into context (silently, don't echo):
- `/root/Purvi/checklist.md`
- `/root/Purvi/Janmam.md`
- `/root/Purvi/flagged.md` — candidate patterns queued by automated wrap-ups
- Today's sync file if it exists: `/root/Purvi/Product sync/DD-MM-YYYY.md`
- The most recent sync file (if different from today) — to see what's still open

### 2 — Surface open to-do's and flagged patterns

Check the most recent sync file for open to-do items (`- [ ]` lines), and `flagged.md` for queued candidate patterns. If any exist, surface them to the user:

> "Open from last sync: [list items]. Flagged since then: [list flags]. Want to review these first?"

Every flagged pattern must be decided during this sync — promoted to Janmam/checklist or dropped — unless the user explicitly defers it. Don't force the ordering, though — if the user has something else to discuss, follow their lead and return to the flags before closing.

### 3 — Tell the user you're ready

One line:

> "Sync loaded. [N open to-do's / No open to-do's, M flagged patterns / no flags]. Go ahead."

## During the sync

**Your role is to listen, capture, and clarify.** The user talks about what happened, what they noticed, what patterns emerged. You:

- Ask clarifying questions when something is ambiguous
- Suggest when something sounds like a Janmam principle ("That sounds like a reusable principle — want to capture it?")
- Track to-do's as they come up
- Don't write to any files yet — hold everything in conversation

**Drafting:** When the user describes something worth capturing, draft it conversationally:
- For Janmam entries: "Here's how I'd write that principle: [draft]. Sound right?"
- For checklist items: "This could be a checklist gate: [draft]. Add it?"
- For sync notes: accumulate discussion points — you'll write them at the end

**Clarify before closing rule applies.** If you have questions, ask them during the sync. Don't carry silent unknowns past "Assembly dispersed."

## Ending the sync

The sync ends when the user says **"Assembly dispersed"** (or close variations).

When you hear it:

### 1 — To-do review

Read back all to-do's — both carried forward from previous syncs and new ones from this conversation:
- Mark completed items
- List open items
- List new items from this sync

Get confirmation before proceeding.

### 2 — Present all writes for approval

Show everything you'd write, grouped by file:

**Sync notes** (`Product sync/DD-MM-YYYY.md`):
> [Draft of today's sync entry — discussion points, decisions, to-do list]

**Janmam** (`Janmam.md`) — only if new principles emerged:
> [Draft of each new entry]

**Checklist** (`checklist.md`) — only if new actionable gates emerged:
> [Draft of each new item and which section it goes in]

**Flag decisions** (`flagged.md`) — for each queued flag, its outcome:
> [Promoted to Janmam/checklist (shown in drafts above) / dropped / explicitly deferred — deferred flags stay in the queue, everything decided is removed]

**Stale items to remove** — if any checklist or to-do items are no longer relevant:
> [What to remove and why]

The user approves, modifies, or rejects each item. **Only write what's approved.**

### 3 — Write and push

Write approved changes, then:

```bash
cd <purvi-path> && git add -A && git diff --cached --quiet || (git commit -m "<short summary>" && git push)
```

Report what was written and whether the push succeeded.

### 4 — Confirm

One line listing what was written and what's still open. Then stop.

## Rules

- **Never write without approval.** Every entry is user-approved.
- **Never create noise.** If the sync had no patterns worth capturing, the sync notes just record what was discussed and the to-do state. No filler.
- **The user drives.** You capture and clarify. Don't steer the conversation toward "productive" topics — if the user wants to talk about something tangential, that's their call.
- **Keep syncs short.** If the conversation is drifting into implementation, flag it: "This sounds like implementation work — save it for a separate session?"
- **Clarify before closing.** Any question raised during the sync must be answered before "Assembly dispersed", unless explicitly deferred.

---
name: update-context
description: This skill should be used when the user asks to "update context", "update logs", "save session notes", "log this session", "update project context", or similar requests to persist what was done in the current conversation. It writes a session narrative to the dated daily log at /root/claude_context/<YYYY-MM-DD>.md and applies targeted updates to the relevant project context at /root/projects_context/<project>/README.md. If the conversation touches a new project that has no folder yet, it asks the user before creating one.
---

# Update Context

Persist this session's work to Nithin's two-tier context system:

1. **`/root/claude_context/<YYYY-MM-DD>.md`** — narrative daily log (always update)
2. **`/root/projects_context/<project>/README.md`** — durable project reference (targeted edits)

**DO NOT write project-specific details to Claude memory files.** All project context — operational notes, design decisions, gotchas, API quirks — belongs in the project README. Memory is reserved for user preferences and feedback only.

| File | Purpose | Style |
|---|---|---|
| Daily log | Chronological narrative of today's session(s) | First-person session voice; includes "why", interim discoveries, every file touched |
| Project README | Current-state operational reference — the single source of truth for project knowledge | Present-tense, as if documenting the system itself; reads like a project handbook |

## Workflow

### Step 0 — Reconcile before write (mid-session invocations)

The conversation that triggered this skill may have already updated some of these files **during the task itself** — e.g. the model edited the README as part of completing the work, then the user invoked `/update-context` to finalise. Treat each write step below as **"bring the file to the right state,"** not **"append the session's narrative."**

For every step that follows:
1. **Read the target file first** to see what's already there.
2. **Compare against the session's outcome.** If a fact is already accurately reflected, skip it. If it's partially there or inaccurate, edit in place. If missing, add it.
3. **Never re-Edit identical content.** Edit calls that just re-write what's already there pollute the diff and mislead future readers about what changed.

The daily log (Step 3) is the most accretive — chronological narrative, less prone to duplicates. The README (Step 4) is the higher-risk surface and warrants a careful re-Read before each Edit.

In the final summary, distinguish files **changed** in this skill invocation from files **already correct** (touched earlier in the conversation, no edit needed). The user wants to know what the skill *added*, not just what's persisted.

### Step 1 — Get today's date
Read `currentDate` from the injected claudeMd context (format `YYYY-MM-DD`). Always use this — do not call `date` via Bash unless the field is somehow missing.

### Step 2 — Identify the project touched in this session

**One project per invocation.** If the session touched multiple projects, pick the primary one (the bulk of the work). If the work is genuinely split, `AskUserQuestion`: "This session touched <A> and <B>. Which should I log under? (Re-invoke `/update-context` for the other after.)"

**Resolve dynamically, not from a hardcoded table.**

1. Run `ls /root/projects_context/` to enumerate current project folders (exclude `README.md`).
2. Scan the conversation for explicit folder names, working directories, file paths, or domain terms. Match against the folder list. Bootstrap hints for known projects (reminders, not authoritative — the folder list is the source of truth):
   - `solar` — `/opt/solar-monitor`, Growatt, ShinePhone, inverter, kWh
   - `events` — `/opt/global-events`, TE scraper, events_calendar
   - `syncthing` — `/etc/syncthing`, drop zone
3. If exactly one folder matches confidently, use it.
4. If zero match or it's ambiguous, **AskUserQuestion** with options drawn from the folder list, plus "New project" and "No project (skip README + memory steps)".

**Sessions with no project** (admin, research, skill/config edits, OS-level tasks): still update the daily log (Step 3), but **skip Step 4 entirely**. The daily log alone is enough — not every session needs durable project state.

### Step 3 — Update the daily log

Path: `/root/claude_context/<YYYY-MM-DD>.md`

- **File does NOT exist:** create with:
  ```
  # Session Notes — <YYYY-MM-DD>

  _Auto-created at HH:MM IST_

  <one-line synopsis of what was worked on>

  ## What we did
  ...
  ```
- **File exists:** append a new continuation section using `Edit` (do NOT overwrite). Format:
  ```
  ---

  # <Project Title> — <descriptor> continuation

  _Started ~HH:MM IST_

  <narrative — what was done, decisions made, surprises, files changed>

  ## Files changed
  - `/path/to/file` — what changed
  ...

  ## Key learnings (optional — only if non-obvious things came up)
  1. ...
  ```

  **End-of-session line.** Only append `_Session ended at HH:MM IST_` if the user clearly signalled the session is over (e.g. "log this and we're done", "wrap up", "save and close"). For mid-session invocations (snapshot calls, "save what we have so far and keep going"), **omit this line** — the session isn't ending and adding a fake closing time misleads future readers. If unsure, omit; appending it later is cheap.

**Descriptor** is a short phrase like "evening continuation", "issue triage", "bug-fix pass", "reconciliation work". Pick something that hints at the session's flavor.

**Times** — use IST. If the conversation has timestamps, derive from those; otherwise reason about start from context and end as "now".

### Step 4 — Update the project README

**Skip this step entirely** if Step 2 resolved "No project" — admin/research/config sessions don't touch a project README.

Path: `/root/projects_context/<project>/README.md`

#### If the project folder does NOT exist:
**ALWAYS** invoke AskUserQuestion before creating anything:
- Question: "This session touched **<inferred-project-name>** but `/root/projects_context/<name>/` doesn't exist yet. Create a new project context folder?"
- Options: "Yes, create README from a starter template" / "No, skip the project README" (label as "Skip" with description "Just update the daily log, leave projects_context alone")

If user chooses Yes:
1. `mkdir -p /root/projects_context/<name>/`
2. Create `README.md` using the starter template below
3. Add a row to `/root/projects_context/README.md` layout table linking to the new folder

Starter template:
```markdown
# <Project> — Project Context

<one-paragraph summary: what it does, where it runs>

**Working code:** `<path>`
**Source of truth on Mac:** `<path or "n/a">`

## Run mechanism
- <systemd? cron? manual?>

## Quick commands
```bash
<key commands>
```

## Account / hardware specifics
- <as applicable>

## Files
| Path | Purpose |
|---|---|
| ... | ... |

## Configuration (.env)
```ini
<key env vars>
```

## Bugs found & fixed
| # | File / line | Bug | Fix |
|---|---|---|---|

## Known operational notes
- ...

## TODO / future work
- ...
```

#### If the project folder exists:

**Targeted edits via `Edit`, not blanket appends.** The README is a reference document; it should read as the *current state*, not a changelog.

##### Gate A — Does this fact belong in the README at all?

A fact crosses the durable threshold only if **all three** pass:

1. **A future session would need this to do its job** — load-bearing, not "nice to know."
2. **It's not derivable from the code itself** — running the script + reading source can't surface it. ("Growatt API returns 0 during faults" passes; "monitor.py uses requests" fails — that's visible to a reader.)
3. **It survives past this session's scope** — general operational knowledge, not a specific incident or in-flight debug session.

If any one fails → daily-log fodder, not README. Skip writing it here.

##### Gate B — Prefer editing existing structure over adding new sections.

New facts attach to existing structure unless the scope is genuinely novel:

- New env var → row in the `.env` block (NOT a new section)
- New API quirk → row in the API-quirks table (NOT a new heading)
- New TODO → bullet under existing TODO section
- New file → row in the Files table
- Configuration / threshold changed → find the existing line, update it
- Bug fixed → row in the "Bugs found & fixed" table
- Operational note → append to "Known operational notes"

Only create a new top-level heading for genuinely novel scope (e.g. when "Backfill" was added as a distinct subsystem). Three new headings in three sessions = red flag — refactor existing structure instead.

##### Prune stale content during the same pass.

The hand that adds new facts also reconciles stale ones. Scan the README for content the current session has invalidated and update or remove it in the same edit:

- TODO items now resolved → strike-through (`~~...~~`) with a one-line note on why, or remove if trivial
- Operational notes contradicted by today's findings → rewrite or remove (don't just add a contradicting note next to the stale one)
- Outdated configuration values, paths, version references → update in place
- "Bugs found & fixed" rows whose understanding has changed → update the row

The README is current-state, not append-only history. If the section is now wrong, fix it; if it's no longer relevant, remove it.

Do NOT just paste daily-log narrative into the README. If the change is *only* a session event with no durable consequence, it belongs in the daily log only.

### Step 5 — Capture in-flight resumption state to NEXT.md (when applicable)

**Skip this step entirely** if Step 2 resolved "No project."

Path: `/root/projects_context/<project>/NEXT.md`

The README captures *durable* project knowledge; `NEXT.md` captures the **in-flight resumption checklist** — what a future session needs to pick up exactly where this one stopped. Daily logs have the narrative; NEXT.md has the curated handoff. Unlike the daily log (date-named, requires remembering when), NEXT.md lives at a well-known path so the next session reliably finds it.

#### When to write NEXT.md (model judgement)

Scan the conversation for unfinished-work signals:
- Explicit TODOs raised this session but not completed
- Mid-refactor state (function partially rewritten, tests added but failing, edits started but not saved)
- Blockers identified but not resolved (waiting on external info, API outage, decision deferred)
- Open design questions deliberately set aside

Decide:
- **Signals clearly present** → write NEXT.md (overwriting any existing one).
- **Signals present but user's wrap-up is ambiguous** (e.g. "save and we'll continue tomorrow" without naming what remains) → `AskUserQuestion`: "There seems to be pending work on `<project>` — want NEXT.md written for resumption?"
- **Session was a clean wrap-up** (user said "done", "ship it", "that's it" — no unfinished signals) → skip writing AND **clear any existing NEXT.md** (delete the file; future sessions should see no pending work).

#### What to write

Use this template. **Omit any section with no content** — empty headings are noise. Order is fixed.

```markdown
_Pending work as of YYYY-MM-DD HH:MM IST_

## Where we left off
<one-paragraph current state — what was just done, what's mid-flight, what the next session is walking into>

## Next steps
1. <concrete first action — specific file, function, command>
2. <next action>
3. ...

## Open questions / blockers
- <question or blocker, with enough context to resume without re-deriving>

## Files mid-edit
- `/path/to/file` (lines N–M) — what's incomplete or what needs to happen there
```

**Be concrete.** Actual file paths, actual line numbers, actual commands to run. A future session must be able to re-enter cold using only NEXT.md + the project README — no scrollback required.

#### Lifecycle

- **Overwrite, don't merge** — the latest session owns the resumption state. If old NEXT.md content is still relevant, the model should include it in the new version explicitly.
- **Clear when resolved** — if the current session completed everything in a prior NEXT.md, delete the file as part of this step. Don't leave stale resumption notes around to mislead future sessions.
- **NEXT.md is not history** — once cleared, it's gone. The narrative of what was once pending lives in the daily logs.

## Step 6 — Purvi pattern flagging (flag only, never write rules)

**The boundary (resolved 10-06-2026):** this automated step only *flags* candidate patterns. It never writes to Janmam, checklist, or sync notes. The dedicated product sync (purvi-sync skill, "Assembly dispersed") is where flagged candidates are reviewed and either promoted to rules or dropped. Flag in passing, decide in person — this prevents unsupervised rule-making and feedback loops.

**Skip this step entirely if:**
- `/root/Purvi/flagged.md` does not exist (Purvi not set up on this machine)
- The context window is tight (conversation has been compressed or is near limit) — prioritise Steps 1–5 over this step

**Purvi repo path:** `/root/Purvi/` (or the user's configured path). This is a ground truth — each machine has its own path. The skill checks existence, not a hardcoded location.

### 6a — Assess and ask

Silently scan the session for candidate patterns:
- Was there a wasted round, a misunderstanding, or a feature built wrong before being corrected?
- Did a new pattern, principle, or decision emerge from the conversation?

**Dedupe before asking:** skim the entry headings in `/root/Purvi/Janmam.md` (and the existing `flagged.md` queue). If the candidate is already captured as a principle or already queued, don't flag it — repeat flags dilute the queue and breed approval fatigue.

**If nothing worth flagging:** skip the rest of Step 6 entirely. Do not mention Purvi, do not comment. Silence means "clean session, nothing to flag."

**If something worth flagging:** ask the user conversationally — e.g.:

> "This session surfaced [brief description]. Flag it for the next product sync?"

On **no** or no response, skip. On **yes**, proceed to 6b.

### 6b — Append approved flags and push

Pull first; if the pull fails (conflict, auth issue), report the error and skip the rest of Step 6. Do not force-push or overwrite.

```bash
cd /root/Purvi && git pull --ff-only 2>&1
```

Append one line per approved candidate under the `## Queue` section of `/root/Purvi/flagged.md` (replace the `(empty)` placeholder if present):

```
- [ ] DD-MM-YYYY — <candidate pattern in one sentence> (context: <which session/project it came from>)
```

Then commit and push:

```bash
cd /root/Purvi && git add -A && git diff --cached --quiet || (git commit -m "Flag: <short candidate summary>" && git push)
```

If the push fails, report the error — do not claim success.

### What NOT to do in this step

- **Don't write rules.** No entries to Janmam, checklist, or sync notes from this step — ever. Those are written only inside a dedicated product sync (purvi-sync skill).
- **Don't flag without approval.** Every flagged.md line is user-approved before it's written.
- **Don't create noise.** If the session has nothing worth flagging, stay silent.
- **Don't echo the checklist.** The checklist is an internal assessment tool, not conversation output.
- **Don't run the "Before building" or "During building" checklist items** — those are for session start (gain-context), not wrap-up.

## Confirming before exit

After all updates, send a brief summary to the user listing:
- Which files were updated and a one-line note on each
- Anything you explicitly chose NOT to do (e.g., "README already correct — no edits needed")

Do NOT ask "is this good?" — just report what was done. The user will redirect if they want changes.

## Style reminders

- Daily log: use the existing voice in `/root/claude_context/2026-05-17.md` as the reference style — informal, second-person to the future reader, includes paths, names files specifically.
- Project README: use the existing voice in `/root/projects_context/solar/README.md` as the reference style — formal handbook tone, tables for structured data, present-tense.

## GitHub backup (after context is written)

After the summary above, offer to back up the affected repo(s) to GitHub (private, under `hobgoblin123`). Ask conversationally — Nithin prefers prose, not the question UI — e.g.:

> "Context updated. Back up **<project>** to GitHub now?"

On **yes**, run these and report their output verbatim:
```bash
/root/github-backup/backup.sh backup-project <project>   # the project's repo (skip if "No project")
/root/github-backup/backup.sh backup claude-workspace    # session logs + skills (always changed)
```
- `<project>` = the `projects_context/` folder name from Step 2 (e.g. `solar`, `events`, `finance_sheet`, `nithins_gallery`, `server-infra`/`syncthing`). For a **No project** session, run only the `claude-workspace` line.
- `backup.sh` runs a secret-scan guard before each commit. **If it prints `ERROR:` / a `✗` line, or the push fails, surface that verbatim and do NOT claim success** — this is the "tell me if backing up breaks things" guarantee.
- **If the `backup.sh` call is blocked by the permission system**, don't fight it — print the two commands above and ask Nithin to run them in his own terminal.
- On **no**, skip silently.

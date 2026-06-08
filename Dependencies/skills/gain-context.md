---
name: gain-context
description: This skill should be used when the user asks to "gain context", "load context", "pull project context", "load <project> context", "what do I know about <project>", "remind me about <project>", or similar requests to surface the durable project reference into the current session. It reads /root/projects_context/<project>/README.md and outputs it so the model has the project's handbook in context. Distinct from update-context, which writes session work out to context files. SessionStart auto-loads today's daily log and the memory index — this skill fills the gap by loading the project README on demand.
---

# Gain Context

Surface a project's durable reference (the README at `/root/projects_context/<project>/README.md`) into the current conversation. The README is Nithin's source of truth for each project — operational handbook, file map, gotchas, configuration. SessionStart does not load it; this skill does.

## Scope

- Loads **the project README only.** Daily logs at `/root/claude_context/<YYYY-MM-DD>.md` are pointers to where files live, not durable knowledge — they're not pulled by this skill.
- **One project per invocation.** Loading multiple muddies the context window. If the user names two, handle the first and ask whether to follow up on the second.
- Pure read. This skill never writes anything (that's `update-context`'s job).

## Workflow

### Step 1 — Resolve the target project

Try in this order; stop at the first match.

1. **Explicit token in the user prompt.** Scan the user's request for any folder name under `/root/projects_context/` (run `ls /root/projects_context/` to enumerate). Case-insensitive match. Example: "load solar context" → `solar`.

2. **Path table by cwd.** If the prompt didn't name a project, check the current working directory against this table:

   | Path / domain mentioned in prompt | Project slug |
   |---|---|
   | `/opt/solar-monitor`, Growatt, ShinePhone | `solar` |
   | `/opt/global-events`, TE scraper, events_calendar | `events` |
   | `/etc/syncthing`, Syncthing drop zones | `syncthing` |

   If cwd or prompt content matches a row, use that slug.

3. **AskUserQuestion.** If neither rule resolved a project, ask: "Which project's context do you want loaded?" with options drawn from `ls /root/projects_context/` (one option per folder, excluding `README.md`).

### Step 2 — Validate the README exists

Check `/root/projects_context/<project>/README.md`.

- **Exists:** continue to Step 3.
- **Folder missing:** the project name is wrong or doesn't have context yet. Fuzzy-match the requested name against existing folder names (simple substring or Levenshtein-ish judgement is fine) and ask via `AskUserQuestion`:
  - If a close match exists: "No `/root/projects_context/<requested>/` exists. Did you mean `<closest>`?" with options for each plausible match plus "None — abort."
  - If no close match: tell the user the project doesn't have a context folder and point them to `/update-context` to create one. Stop.

### Step 3 — Stat the README and produce a freshness header

Get the README's mtime (e.g. `stat -c %Y <path>` or read the file's modification date). Compute days since modification.

Output a one-line header *before* loading the README content:

```
_Project README last updated YYYY-MM-DD (N days ago)._
```

If > 14 days, append:

```
 ⚠ potentially stale — verify against current code before trusting.
```

### Step 4 — Read the README into context

Use the Read tool on `/root/projects_context/<project>/README.md`. Do not transform, summarise, or filter — the README is already the curated, durable view. Letting the full content land in the conversation puts it in the model's working context.

### Step 4b — Surface pending work from NEXT.md (if present)

Check `/root/projects_context/<project>/NEXT.md`.

- **Not present, or present but empty** → continue to Step 5 with no flag.
- **Present with content** → Read it (so its content lands in conversation context like the README). Note its presence so Step 5's confirmation line can flag it prominently.

NEXT.md is the prior session's resumption checklist (written by `update-context`'s Step 6). Surfacing it ensures the user's next move is informed — either they want to resume the pending work, or they have a fresh ask but should at least know what's outstanding.

### Step 4c — Load ground truths and Purvi checklist

**Ground truths** — If `/root/.claude/ground-truths.md` exists, read it into context. This is the environment's `.env` — device-specific operational facts (paths, auth, services). It is local-only, never backed up or committed to any shared repo. Each machine has its own copy.

**Purvi checklist** — If the Purvi path from ground truths exists (or default `/root/Purvi/checklist.md`), read it into context alongside the project README. This loads the product quality gates so the session starts with the right lens. The "Before building" section is the one that matters at session start: ground truth check, value check, clarifying question, pattern vs scope.

Do not echo the ground truths or checklist to the user — just load them silently.

**If Purvi is not found** — tell the user: "Purvi is not installed on this machine. Product quality gates won't load this session." One line, then continue with the rest of the skill as normal.

**Skill diff check** — If ground truths has a `Skill diff check frequency` and a `Last skill diff check` date, check if a diff is due based on the frequency (daily/weekly/monthly/yearly/never). If due:

1. Compare each file in `<purvi-path>/Dependencies/skills/` against the corresponding live skill file (matching by filename → skill name → `~/.claude/skills/<name>/SKILL.md` or plugin cache path).
2. If any differ, tell the user: "Skill snapshots in Purvi are out of date: [list changed skills]. Update the snapshots?"
3. On yes, copy the live versions to `Dependencies/skills/`, commit, push.
4. Update `Last skill diff check` in ground truths to today's date.

If frequency is `never`, skip entirely. If not due yet, skip silently. Don't block on it.

If the user said "Start Session" (rather than naming a specific project), resolve the project from context as usual (Step 1), then load the project README, ground truths, and Purvi checklist. "Start Session" is an alias for "gain context + load product gates."

### Step 5 — Confirm to the user

One line. If NEXT.md was not present:

```
Loaded `solar` context (README, 203 lines, last updated 2026-05-17).
```

If NEXT.md was present and loaded:

```
Loaded `solar` context (README, 203 lines, last updated 2026-05-17). ⚠ NEXT.md present — pending work from a prior session is in context.
```

Do NOT echo the README or NEXT.md content back to the user as a summary — the user can see what was loaded from the Read tool results, and re-summarising bloats the conversation. Just confirm and stop.

## Style reminders

- Stay terse. Skill output should be: optional freshness header → Read tool call → one-line confirmation. Nothing else.
- If the user asked a follow-up question alongside the load request ("load solar context and tell me what the panel kWp is"), answer it *after* the load using the now-loaded README content.
- Never load multiple projects in one invocation. If user wants two, do one, confirm, then ask whether to load the second.
- If the user re-invokes for a project already loaded in this session, just confirm — the README is already in context, no need to re-Read unless the file changed since.

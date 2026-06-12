# Purvi — agent entry point

You are reading the repo of a product-management toolkit for AI coding assistants. If you are an AI agent (any platform — Codex, Aider, Copilot, Gemini, Cursor, Claude, or other), this file tells you how to use it.

## Apply immediately

1. Read `checklist.md` — these are the gates. Keep the "Before building" section in mind from the start of every session; run "During/After building" at the matching phase. They are plain rules; no tooling needed.
2. Read `Janmam.md` when a gate needs its reasoning — every gate traces to a principle born from a real failure.
3. Never write rules directly. If you notice a reusable pattern, append a one-line candidate to `flagged.md` (format documented in the file) and leave the decision to a human-driven product sync.

## The sync protocol

A "product sync" is a conversation, not a tool: load `checklist.md`, `Janmam.md`, `flagged.md` and the latest `Product sync/` note → surface open to-dos and flags → the user drives, you capture → every flag is promoted (Janmam + gate if actionable) or dropped → nothing is written without explicit approval → notes go to `Product sync/DD-MM-YYYY.md`. Full reference: `Dependencies/skills/purvi-sync.md`.

## Reading the skill snapshots on other platforms

`Dependencies/skills/*.md` are verbatim snapshots of Claude Code skills. The *workflows* are portable; only the tool names are Claude-specific. Translate:

| Claude Code tool | Generic equivalent |
|---|---|
| `Read` / `Write` / `Edit` | read file / write file / edit file |
| `Bash` | run shell command |
| `Grep` / `Glob` | search file contents / find files by pattern |
| `Agent` (subagent) | spawn a sub-task, or do it inline if your platform has no subagents |
| `Skill` tool invocation | open the named snapshot in `Dependencies/skills/` and follow it |
| `AskUserQuestion` | ask the user in plain prose |
| `TodoWrite` / Task tools | your platform's task list, or a markdown checklist |

## Companion tools (optional, multi-platform)

- **ponytail** (code minimalism, enforces the "Minimal-code ladder" gate): https://github.com/DietrichGebert/ponytail — ships native rules files for Cursor, Windsurf, Cline, Copilot, Aider; Claude Code installs it as a plugin. Local overrides for this toolkit: stable external dependencies are acceptable, and the ladder never waives TDD.
- **graphify** (knowledge graphs, enforces the graph-first "Ground truth check"): https://github.com/safishamsi/graphify — supports 15+ assistants. Graphs (`graphify-out/`) are machine-local; never commit them.

## Machine-local, do not expect in this repo

Ground truths (device facts: paths, services, auth locations) live in a local file outside any repo — recreate per machine. See README "Ground truths" section.

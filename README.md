# Purvi

Purvi is a product management toolkit for AI coding assistants. It helps them build the right thing — not just build things right.

## The problem

AI coding assistants are great at writing code. But they're terrible at knowing *what* to write. Ask one to "make this editable" and it'll happily build a feature that already exists somewhere else. Ask it to "add a toggle" and it'll over-engineer a whole new component when three lines would do. The code works. It just wasn't what you needed.

Every wrong turn burns tokens — the AI's thinking budget. Wasted tokens mean wasted time, wasted money, and wasted back-and-forth where you explain what you actually meant.

## The idea

What if the AI asked the right questions *before* writing code?

That's what Purvi does. It's a set of principles and checklists — born from real product conversations — that teach AI assistants to think like a product manager. Before building anything, check: Does this already exist? What new value does this add? Am I solving the right problem?

## How it works

Purvi has four parts:

### 1. Checklist (`checklist.md`)

A simple list of questions to run through before, during, and after building anything. Things like:

- Does this add something new, or am I duplicating an existing feature?
- Am I designing for mobile first?
- If I can't see the result, am I saying so instead of pretending it works?

### 2. Janmam (`Janmam.md`)

A running list of principles. Each one traces back to a real conversation where something went wrong, and the reflection that followed. Not theory — lessons learned the hard way.

For example: *"If someone says 'I like how X works,' they mean the interaction style, not 'build an exact copy of X.'"* That came from a real session where the AI cloned a 3-field edit form when the user wanted a full 7-field editor.

### 3. Flagged patterns (`flagged.md`)

A queue, not a rulebook. Automated session wrap-ups may *flag* candidate patterns here, but nothing becomes a rule until a deliberate, human-driven product sync promotes it. **Flag in passing, decide in person** — this keeps the rule set intentional and stops the system from amplifying its own output. Every sync empties the queue: each flag is promoted (to Janmam, plus a checklist gate if actionable) or dropped.

### 4. Product sync notes (`Product sync/`)

Dated notes from product conversations. This is where new principles are born. The pattern looks like this:

1. Something goes wrong during a build
2. The user asks: "What could you do better next time?"
3. The wrap-up flags the candidate pattern into `flagged.md`
4. At the next product sync, the flag is reviewed — promoted into Janmam, or dropped
5. If it's actionable, it also becomes a checklist gate — a principle that keeps getting violated *must* become a gate; enforcement beats intention

The sync notes are the raw material. Janmam is the refined output. The checklist is the daily tool. (`Dependencies/skills/purvi-sync.md` describes how a sync conversation runs; `Product sync/instructions.md` covers the note-file conventions.)

## Why "token prioritization"?

AI models have a thinking budget (tokens). Today's models often spend that budget on the wrong things — building before understanding, guessing instead of asking, reworking instead of getting it right the first time.

Purvi makes every token count by pointing the AI in the right direction before it starts working. One good question at the start saves hundreds of lines of wasted code.

As models get more capable, this matters more, not less. A smarter model that builds the wrong thing just builds the wrong thing faster.

## Setting up Purvi on your LLM

Purvi works by hooking into your AI assistant's session lifecycle — loading the checklist when you start working, and reviewing it when you wrap up. Here's how to set it up.

### Step 1 — Fork and clone

```bash
# Fork this repo on GitHub, then:
git clone https://github.com/<your-username>/Purvi.git ~/Purvi
```

Put it wherever makes sense for your setup. The path just needs to be consistent — your skills will reference it.

### Step 2 — Wire into your session start

Your AI assistant needs to load `checklist.md` at the start of every session. How you do this depends on your tool:

**Claude Code** — Edit your `gain-context` skill (or equivalent session-start skill) to add a step that reads `~/Purvi/checklist.md` into context. The "Before building" section is what matters at session start. See `Dependencies/skills/gain-context.md` for a reference implementation.

**Other LLMs** — If your tool supports custom instructions, system prompts, or skills, add a rule: "At session start, read `~/Purvi/checklist.md` and keep the 'Before building' items in mind for this session."

### Step 3 — Wire into your session wrap-up

When you finish a session, the checklist's "After building" and "Wrapping a sync" sections should run. This is where reflection happens — did something go wrong? Is there a principle worth capturing?

**Claude Code** — Edit your `update-context` skill to add a step before the final confirmation that reads `~/Purvi/checklist.md`, checks the "After building" items, and flags candidate patterns into `flagged.md` (never writing rules directly). See `Dependencies/skills/update-context.md` for a reference implementation (look for "Step 6 — Purvi pattern flagging").

**Other LLMs** — Add a wrap-up rule: "Before closing, check `~/Purvi/checklist.md` 'After building' section. Flag anything relevant."

### Step 4 — Start using it

The loop works like this:

1. **Start a session** → checklist loads → you build with the right lens
2. **Something goes wrong** → you ask "what could you do better next time?"
3. **Wrap up** → the checklist catches anything you missed; candidate patterns get flagged into `flagged.md` (flag only — no rule-writing outside a sync)
4. **Next product sync** → each flag is promoted to `Janmam.md` (plus a `checklist.md` gate if actionable) or dropped, until the queue is empty
5. **Next session** → the improved checklist loads → repeat

Run product syncs (`Product sync/` folder) periodically — and *before spec'ing any new feature or project*, not just retrospectively. The sync notes are the raw material; Janmam is the distilled output. One watchdog worth keeping: if several sessions pass with an empty flag queue, say so out loud — silent flagging failure looks identical to clean sessions.

### Step 5 — Make it yours

Purvi is a starting point, not a prescription. The principles in Janmam came from one person's workflow — yours will be different. Delete what doesn't apply, add what does. The structure (checklist → principles → sync notes → feedback loop) is the valuable part, not the specific entries.

## Using Purvi with another LLM

Short answer: yes — point any capable LLM at this repo and the principles work immediately. The longer answer is that Purvi has three layers with different portability:

1. **The principles and gates (`checklist.md`, `Janmam.md`) — fully portable.** They're plain markdown. Paste them into custom instructions, a system prompt, a `GEMINI.md`/`AGENTS.md`, or just say "read checklist.md and Janmam.md from this repo and apply the gates to everything we build." No tooling required.
2. **The feedback loop (flagging, syncs) — portable as a protocol.** `Dependencies/skills/purvi-sync.md` describes how a sync conversation runs (load context → surface flags → user drives → all writes approved → queue emptied); any LLM can follow it conversationally. The automation (auto-loading at session start, auto-flagging at wrap-up) needs whatever your platform offers — Claude Code uses skills (reference implementations in `Dependencies/skills/`), other tools use rules files or custom instructions. Worst case, you load the checklist manually at session start; the loop still works, just with more discipline on your side.
3. **The tool pair — ships its own multi-platform support.** ponytail provides rules files for Cursor, Windsurf, Cline, Copilot, and Aider in its repo; graphify supports 15+ assistants. Install per their docs and the "Minimal-code ladder" and graph-first ground-truth gates apply unchanged.

What does *not* port: your ground truths file (machine-specific by design — recreate per environment) and the `Dependencies/skills/` snapshots (Claude Code reference implementations — read them for the *pattern*, not to run them elsewhere).

### Going further: full cross-LLM continuity on one machine

If you run multiple LLMs on the same server (e.g. Claude Code for one session, Codex or Gemini CLI for the next), you can make handoffs seamless by building a **project map** — a single markdown file that any LLM reads cold and gets the full picture. Here's what we built for our setup (adapt paths to yours):

1. **`PROJECT_MAP.md`** — a master pointer file listing every project (path, GitHub repo, knowledge-graph stats), every context source (ground truths, project handbooks, daily logs, resumption notes, graphs, Purvi gates), and a numbered "Start here" sequence. An LLM reads this one file and knows where everything lives.

2. **`PREFERENCES.md`** — your working style, communication rules, and building constraints, extracted from whatever memory/preference system your primary LLM uses. This makes your preferences portable: Claude knows them from its memory; Codex learns them from this file.

3. **In-flight handoff via `NEXT.md`** — the session wrap-up skill writes a resumption checklist to `<project>/NEXT.md` when work is left mid-stream. The next LLM (any platform) reads it and picks up exactly where the last session stopped.

4. **Shared knowledge graphs** — `graphify-out/` in each project root. Any LLM on the same machine can query them directly (`graphify query "..."`) instead of re-exploring the codebase from scratch.

5. **Auto-maintenance** — the session wrap-up skill (reference: `Dependencies/skills/update-context.md`, Step 7) checks both the project map and the preferences file on every wrap-up, updating stale rows and adding new feedback. This keeps the handoff documents current without manual effort.

Put the map and preferences file somewhere that syncs across your devices (we use a Syncthing folder) so you have a copy even if the server is down. The map points at everything; the files it points at do the real work. The key insight: Purvi's principles and gates are the shared layer (this repo), but the *operational glue* — where things live, who you are, what's in flight — lives in these local documents that any LLM can read.

## Ground truths — your `.env` file

Ground truths are device-specific operational facts: auth paths, service configs, credential locations, installed tools, port numbers. They're the equivalent of a `.env` file — local to the machine, never committed to a shared repo.

**Why separate?** Purvi documents *principles* ("check what's already in place before setting up"). Ground truths hold *facts* ("GitHub auth is at `/path/to/credentials`"). The principle is universal. The facts are not. Mixing them makes the facts look portable when they aren't, and the next person on a different machine acts on stale info.

**Where to put them:** Create a `ground-truths.md` file in your local AI config directory (e.g., `~/.claude/ground-truths.md` for Claude Code). This file should:

- **Never be committed** to any git repo — it's outside your project directories
- **Never be backed up** to GitHub or any shared location
- **Be loaded at session start** — wire your session-start skill (like `gain-context`) to read it into context
- **Hold only environment-specific facts** — paths, auth, services, hardware. Not principles, not preferences, not project knowledge.

Example structure:

```markdown
# Ground Truths

## Auth
- GitHub PAT: `/path/to/credentials`
- API keys: `/path/to/.env`

## Services
- Backend: port 8000, systemd unit `my-backend.service`
- Frontend: port 3000, WorkingDirectory=/path/to/app

## Paths
- Projects: `/home/user/projects/`
- AI skills: `~/.claude/skills/`
```

The relationship between Purvi and ground truths is the same as code to `.env`: the code says "read the config from the environment"; the `.env` says what the config actually is on *this* machine.

## Starting work under Purvi

Purvi activates through your session-start skill — but the flow differs for existing vs brand-new projects.

### Existing project

Say **"gain context"** (or "Start Session"). The session-start skill loads everything in one pass:

1. The project's context README (durable project knowledge)
2. Local ground truths (machine-specific facts)
3. `Purvi/checklist.md` — the gates, including the tool gates: ponytail's **minimal-code ladder** ("During building") and graphify's **graph-first ground-truth check** ("Before building")

No per-project setup is needed for the tools themselves: ponytail is installed once as a plugin and applies everywhere; graphify kicks in automatically wherever a `graphify-out/` knowledge graph exists — codebase questions route through the graph instead of re-exploring files.

**Spec'ing a new feature? Start a sync first.** Before building anything non-trivial on an existing product, run a product sync (purvi-sync skill) to spec it: what new value it adds, which existing design rules it touches, what the constraints are. This is where the full toolkit engages in the right order — the sync runs "Value check" and "Design-rule tension check" against the product's principles, graphify answers "what already exists" from the knowledge graph, and the build session that follows applies ponytail's ladder to what survives the spec. Skipping the sync means the gates only catch problems mid-build, when they're expensive; the sync catches them before a line of code exists.

### New project

There's no context to load yet, so activation is a bootstrap sequence:

1. **Spec it in a product sync** (purvi-sync skill). What is it, what new value does it add, what are its constraints and design rules? The sync notes become the project's first product record — this is where "Value check" and "Design-rule tension check" run for the very first time, before any code exists.
2. **Start building under the gates.** The checklist loads at session start regardless of project age — "gain context" works from session one (the checklist doesn't depend on a project README existing). Ponytail's ladder applies from the first line of code.
3. **Create the project context at first wrap-up.** The session wrap-up skill (update-context) offers to create the project's context folder/README on first use. From then on, "gain context <project>" resolves it like any existing project.
4. **Build the knowledge graph once there's code to map.** Run `/graphify .` in the project root. From that point the Ground-truth gate queries the graph (`graphify query "<question>"`) instead of grepping, and keeps it fresh with `/graphify . --update`. **Keep `graphify-out/` local** — add it to the project's `.gitignore`. The graph is a derived artifact that can embed environment facts (paths, service names, doc excerpts), and it regenerates on demand; like ground truths, it belongs to the machine, not the shared repo.

The loop after bootstrap is the same as for existing projects: sync to spec → "gain context" to build under the gates → wrap-up flags patterns → next sync refines the rules.

## Dependencies

The `Dependencies/skills/` folder contains reference snapshots of skills that Purvi integrates with. These are copies — the live versions evolve independently. Use them to understand how Purvi hooks into a session lifecycle, then adapt to your own setup.

Beyond the session-lifecycle skills, two external tools are part of the toolkit (snapshots in the same folder):

- **ponytail** ([DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)) — code minimalism. Enforcement arm of the "Minimal-code ladder" gate: Purvi decides *whether* to build; ponytail keeps *what gets built* minimal. See `Dependencies/skills/ponytail.md` for the ladder and the local overrides.
- **graphify** ([safishamsi/graphify](https://github.com/safishamsi/graphify)) — knowledge graphs. Enforcement arm of the "Ground truth check" gate: when a project has a `graphify-out/` graph, query it before re-exploring. See `Dependencies/skills/graphify.md`.

## Status

Early stage. The principles and checklists grow with every product conversation. Nothing here is hypothetical — every entry comes from a real session where something was built, something went wrong, and something was learned.

## License

MIT

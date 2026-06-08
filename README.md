# Purvi

Purvi is a product management toolkit for AI coding assistants. It helps them build the right thing — not just build things right.

## The problem

AI coding assistants are great at writing code. But they're terrible at knowing *what* to write. Ask one to "make this editable" and it'll happily build a feature that already exists somewhere else. Ask it to "add a toggle" and it'll over-engineer a whole new component when three lines would do. The code works. It just wasn't what you needed.

Every wrong turn burns tokens — the AI's thinking budget. Wasted tokens mean wasted time, wasted money, and wasted back-and-forth where you explain what you actually meant.

## The idea

What if the AI asked the right questions *before* writing code?

That's what Purvi does. It's a set of principles and checklists — born from real product conversations — that teach AI assistants to think like a product manager. Before building anything, check: Does this already exist? What new value does this add? Am I solving the right problem?

## How it works

Purvi has three parts:

### 1. Checklist (`checklist.md`)

A simple list of questions to run through before, during, and after building anything. Things like:

- Does this add something new, or am I duplicating an existing feature?
- Am I designing for mobile first?
- If I can't see the result, am I saying so instead of pretending it works?

### 2. Janmam (`Janmam.md`)

A running list of principles. Each one traces back to a real conversation where something went wrong, and the reflection that followed. Not theory — lessons learned the hard way.

For example: *"If someone says 'I like how X works,' they mean the interaction style, not 'build an exact copy of X.'"* That came from a real session where the AI cloned a 3-field edit form when the user wanted a full 7-field editor.

### 3. Product sync notes (`Product sync/`)

Dated notes from product conversations. This is where new principles are born. The pattern looks like this:

1. Something goes wrong during a build
2. The user asks: "What could you do better next time?"
3. The reflection produces a principle
4. The principle goes into Janmam
5. If it's actionable, it becomes a checklist item

The sync notes are the raw material. Janmam is the refined output. The checklist is the daily tool.

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

**Claude Code** — Edit your `update-context` skill to add a step before the final confirmation that reads `~/Purvi/checklist.md` and checks the "After building" items. See `Dependencies/skills/update-context.md` for a reference implementation (look for "Step 6 — Purvi product checklist").

**Other LLMs** — Add a wrap-up rule: "Before closing, check `~/Purvi/checklist.md` 'After building' section. Flag anything relevant."

### Step 4 — Start using it

The loop works like this:

1. **Start a session** → checklist loads → you build with the right lens
2. **Something goes wrong** → you ask "what could you do better next time?"
3. **The reflection produces a principle** → add it to `Janmam.md`
4. **If it's actionable** → add it to `checklist.md`
5. **Wrap up** → the checklist catches anything you missed
6. **Next session** → the improved checklist loads → repeat

Run product syncs (`Product sync/` folder) periodically to capture patterns from multiple sessions. The sync notes are the raw material; Janmam is the distilled output.

### Step 5 — Make it yours

Purvi is a starting point, not a prescription. The principles in Janmam came from one person's workflow — yours will be different. Delete what doesn't apply, add what does. The structure (checklist → principles → sync notes → feedback loop) is the valuable part, not the specific entries.

**Ground truths are local.** Things like auth paths, server configs, and tool setups are facts about *your* environment. Store those in your local skills database, not in this repo. The repo documents the *principle* ("check what's already in place"); the *facts* live locally.

## Dependencies

The `Dependencies/skills/` folder contains reference snapshots of skills that Purvi integrates with. These are copies — the live versions evolve independently. Use them to understand how Purvi hooks into a session lifecycle, then adapt to your own setup.

## Status

Early stage. The principles and checklists grow with every product conversation. Nothing here is hypothetical — every entry comes from a real session where something was built, something went wrong, and something was learned.

## License

MIT

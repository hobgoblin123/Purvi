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

## Status

Early stage. The principles and checklists grow with every product conversation. Nothing here is hypothetical — every entry comes from a real session where something was built, something went wrong, and something was learned.

## Want to use this?

Purvi is designed for anyone working with AI coding assistants. Drop the checklist into your workflow, reference Janmam when you're not sure if the AI understood you, and run a product sync when you notice patterns worth capturing.

## License

MIT

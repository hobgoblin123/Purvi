# Janmam

Running principles born from product syncs. Reference this before building anything.

---

## 08-06-2026 — Duplication is not value

**What happened:** Asked to make recent ledger items editable. Built a copy of the existing 3-field edit sheet (category, expenses, income) — the exact same feature that already existed in the spending history overlay. Added zero new capability.

**What was actually needed:** Full field editing (date, particulars, expenses, income, mode, category, notes) — something that didn't exist anywhere. The reference to the spending history edit was about the *interaction pattern* (bottom sheet, tap to edit), not the scope.

**The principle:** Before building, ask — *does this already exist somewhere?* If yes, you're misunderstanding the ask. The user is asking for something new, not a copy. Features that add no new capability add no value.

**The question that would have saved a round:** "Do you want the same 3-field edit, or the full entry form in that bottom sheet?"

### Takeaways

1. **"I like what we did for X" = the pattern, not the scope.** When someone references an existing feature, they're pointing at the UX model, not asking you to clone it.
2. **If it already exists, you're wrong.** A feature request that results in a duplicate means you misread the intent. Step back and ask what's *new*.
3. **One clarifying question beats one wasted round.** The cost of asking is one message. The cost of building the wrong thing is a full implementation cycle plus the rework.
4. **Value test:** Before starting, state in one sentence what new capability this adds. If you can't, or if the answer is "the same thing but in a different place," stop and re-examine.

---

## 08-06-2026 — "What can you do better?" is the highest-value question

**What happened:** The same question — "what can you do next time to get here quicker?" — has now been asked twice across different projects (gallery mute toggle, finance sheet edit). Both times it turned a one-off mistake into a reusable principle.

**The principle:** The fix solves one instance. The reflection question produces a rule that prevents a class of future mistakes. This is the core loop that Purvi is built on: observe failure → reflect → capture principle → apply forward.

---

## 08-06-2026 — Always mobile first

**What happened:** External advice received — always be mobile friendly, always be a mobile first user.

**The principle:** Mobile is the default lens, not a follow-up check. Design for the smallest screen first. If it works on mobile, it works everywhere. If you only checked desktop, you checked half the product.

**How to apply:** Every UI change starts with "how does this look on a phone?" before "how does this look on a monitor?"

---

## 08-06-2026 — Ground truth check

**What happened:** Needed GitHub auth to create a repo. Installed `gh` CLI, asked the user to authenticate interactively — then discovered credentials were already stored in a git credential file. The answer was already there. We just didn't check.

**The principle:** Before setting up anything, check what's already in place. When you discover something critical, write it down immediately. Knowledge that isn't written down isn't knowledge — it's a landmine waiting to waste someone's time.

**The rule:** If it's ambiguous, don't send it. The goal is zero ambiguity in everything — code, communication, documentation. If something could be misunderstood, unclear, or forgotten, make it explicit before moving on.

**Ground truths are your `.env` file.** Device-specific operational facts — auth paths, service configs, installed tools, credential locations — are not portable knowledge. They belong in a local ground truths file (e.g., `~/.claude/ground-truths.md`) that is:
- **Local only** — never committed to any shared repo, never backed up to GitHub
- **Loaded at session start** — the gain-context skill reads it into context alongside the project README
- **Per-machine** — each environment has its own copy with its own facts

The shared repo (Purvi) documents the *principle* — "check what's already in place, write it down." The ground truths file holds the *facts*. This is the same relationship as code to `.env`: the code says "read the database URL from config"; the `.env` says what the URL actually is on this machine.

**How to apply:** Before any setup or configuration step, ask "is this already done?" Before moving on from any discovery, ask "will I or someone else need this again?" If yes, and it's environment-specific, write it to the local ground truths file. If it's a universal principle, write it to Janmam. Never mix the two.

---

## 08-06-2026 — Clarify before closing

**What happened:** Questions raised mid-sync were carried out silently — not answered, not explicitly deferred. Silent unknowns leak out of syncs and get lost.

**The principle:** Every question raised during a sync must be resolved before the sync ends. The only exception is an explicit deferral — "we'll clarify that later." If it wasn't said, it wasn't deferred, and it needs an answer now.

**How to apply:** Before wrapping any sync, scan back for unanswered questions. If one exists, surface it. Don't close with silent unknowns.

---

## 09-06-2026 — Don't assume, always check

**What happened:** Asked whether git worktrees would help efficiency. Gave a confident answer listing benefits and edge cases — without checking the actual repos. User pushed back: "Don't assume always check and clarify." Inspecting the real state revealed hardcoded systemd paths, 288MB duplication per worktree, and missing secrets — real blockers that were missed.

**The principle:** Never answer "will X work for us?" from general knowledge. Check the actual environment first. A tool that's great in general can be useless or harmful in a specific setup. This is ground truth applied to decision-making, not just setup.

**How to apply:** When evaluating any tool, pattern, or approach — check the real state before recommending. Run the commands, read the configs, inspect the files. "It generally works well" is not an answer to "will it work here?"

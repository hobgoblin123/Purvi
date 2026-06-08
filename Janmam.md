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

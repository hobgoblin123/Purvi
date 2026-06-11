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

---

## 09-06-2026 — Don't duplicate across context systems

**What happened:** Product sync wrap-up was about to write sync notes that repeated everything already in the daily log. User caught it: "All of these contexts are there in claude_context daily log."

**The principle:** Each context system has a job. Daily logs capture session narrative. Sync notes capture *patterns and principles* born from sessions — not the session itself. If a sync produced no new patterns, the sync file has nothing to add. Writing "here's what we did" in both places creates maintenance burden and dilutes signal.

**How to apply:** Before writing sync notes, ask: "Is there anything here that the daily log doesn't already cover — a new principle, a pattern, a decision?" If no, skip. Sync notes earn their existence by distilling, not by echoing.

---

## 10-06-2026 — Verify findings before acting on them

**What happened:** Ran a server security audit via parallel subagents. Three headline findings were false positives: a "expired" Tailscale key that wasn't (the field was `None`/disabled, not expired), "exposed" API docs that were actually gated (302/404 through nginx), and "unauthenticated" finance routes that nginx protects via `auth_request`. Each was wrong because it judged one layer in isolation.

**The principle:** An audit or review finding is a hypothesis, not a fact. Before acting on it, reproduce it against the running system — especially for anything that spans layers (a proxy in front of an app, TLS terminating upstream, a config field that can mean "disabled"). Automated or delegated analysis sees fragments; the truth is in how the layers compose.

**How to apply:** For each finding, reproduce it live before fixing — curl the actual URL, read the actual config, check the actual field value. If a fix's premise can't be reproduced, the finding is suspect. Judge the system, not the file. (Pairs with "Don't assume, always check.")

---

## 10-06-2026 — Flag in passing, decide in person

**What happened:** The automated sync step in update-context and the dedicated product sync had an unresolved boundary — both could plausibly write rules, risking unsupervised rule-making and feedback loops.

**The principle:** Observation and decision are separate roles. Automated processes may *flag* candidate patterns, but only a deliberate human-driven review promotes them to rules. This keeps the rule set intentional and prevents the system from amplifying its own output.

**How to apply:** update-context appends candidates to `flagged.md` (approval still required). purvi-sync surfaces them at the next sync, where each is promoted to Janmam/checklist or dropped.

---

## 11-06-2026 — Verify end-to-end as the real user/role

**What happened:** Built the same Web Push pipeline for three apps. On cycle the VAPID key was generated as root while the service runs as user `cycle`, so the key was unreadable → the enrol endpoint 500'd → "Setup failed" on Nithin's phone. It stayed invisible because I only tested key *generation* (as root, which can read anything) and the backend unit tests — never the actual enrol flow as the service user or in a real browser. The same build also diverged across the three (cron as root vs app user, key ownership) — a direct violation of "Consistency is a feature" the day after it was written.

**The principle:** "Tests pass" and "it works" are different claims. A feature with auth, file permissions, or multiple surfaces is only verified when exercised through its real path — as the unprivileged service user, in a real client — not as root and not by unit tests alone. Root reads every file, so it masks exactly the permission bugs that bite in production. And when you build one capability N times, the instances must match; divergence is where the bugs hide.

**Why a gate, not just a principle:** "Consistency is a feature" already existed as a principle and I violated it the next day — proof that a principle without an enforcement step gets ignored. The lesson became a checklist gate ("Verify end-to-end as the real user/role"), because the gate is what forces the check. When a principle keeps getting violated, the fix is a gate.

**How to apply:** Before calling done — run the real flow as the service user (`sudo -u <app> …`) and in a real browser/device; if built across services, diff ownership/service-user/cron-user/paths. Corollary: when you find one instance of a bug, audit the whole class immediately (the cycle key fix surfaced the cron-user and stray-test-data issues across the other pipelines). Pairs with [[verify findings before acting]] and "Consistency is a feature".

---

## 11-06-2026 — A change's blast radius includes shared infrastructure

**What happened:** Self-hosting cycle's fonts + Chart.js (to make the PWA work offline) took a page load from ~10 same-origin requests to ~16. Those assets used to load from Google/jsdelivr — cross-origin, so they never counted against our nginx rate limit. Same-origin now, they do. A cold load exceeded the `api` zone's `burst=10` and nginx 429'd the dashboard API call, which returned an HTML error page → `res.json()` threw → the app showed no data. The change was correct in isolation; it broke through shared infrastructure.

**The principle:** A change isn't scoped to the file you edited. Anything that alters request count, request paths, or resource load lands on shared infrastructure — rate-limit zones, CPU/memory cgroups, connection pools — that other features and other apps also depend on. Reason about the change at the level of the shared resource, not just the component.

**How to apply:** When a change adds same-origin assets, changes how many requests a page fires, or shifts load, check it against the shared limits before deploying (nginx `limit_req` zones/burst, cgroup caps). Rate limits belong on the API, never the static catch-all — and a 429/500 returning HTML reads to the client as "no data," not "rate limited," so it hides as a different bug.

---

## 10-06-2026 — Cookieless fetches are part of the logged-out surface

**What happened:** Iris shipped with an unstyled login page — the auth middleware 302'd CSS for anonymous users, and nobody had ever loaded the page logged-out during dev. Tonight's PWA rollout hit the same class proactively: manifest fetches and SW update checks run without cookies, so all three apps got explicit auth exemptions plus logged-out regression tests before deploy.

**The principle:** An auth-gated app still has a logged-out surface: login-page assets, PWA manifests, service workers, icons — anything the browser fetches without a session. That surface must be deliberately public, and deliberately *small*: static app-identity assets only (test: "fine on the front page?"). Anything carrying user data stays gated — if a cookieless client needs data (widgets), use a token guard, never a blanket exemption.

**How to apply:** Whenever adding assets the browser fetches on its own, exempt them explicitly and add a logged-out regression test (assets 200 anonymously, app pages still redirect). Test the logged-out experience with a real browser — authed-dev curl checks never catch it.

---

## 10-06-2026 — Consistency is a feature

**What happened:** Two scales of the same lesson in two days. Across projects: the PWA rollout replicated the iris pattern verbatim in three apps — same file names, manifest shape, SW policy, exemption style. Within an app: iris's widget and dashboard disagreed on streak ordering until both were derived from one rule (`streak_started_at DESC`), and the dashboard's day-0 tickers were reformatted to H:MM:SS so widget and dashboard read identically.

**The principle:** Consistency is itself a capability, at both scales. Across projects, one mental model for N apps means faster sessions, bugs found once are fixed everywhere, and diffs stay reviewable. Within an app, every surface showing the same data must agree — same rule, same ordering, same format. Two surfaces that disagree read as a bug even when both are individually "correct." Divergence is allowed, but it must be a decision, not a drift.

**How to apply:** Before building a capability a sibling project already has, copy its implementation shape and learnings — diverge only with a stated reason. When two surfaces present the same data, derive one shared rule from the user's expected example and apply it to every surface, presentation format included. Match each repo's local conventions when adding code to it.

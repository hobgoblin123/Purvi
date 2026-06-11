# Product Checklist

Run through this on every product sync and every session wrap-up (update-context).

## Before building

- [ ] **Ground truth check** — Before setting up, connecting, or configuring anything, check what's already in place. Before moving on from a discovery, write it down. If it's ambiguous, don't send it. Ground truths are environment-specific — store them in the local skills database, not in shared repos.
- [ ] **Value check** — Does this add a new capability? If it already exists somewhere, you're misunderstanding the ask.
- [ ] **One clarifying question** — State what you think the user wants in one sentence. If you're not sure, ask before writing code.
- [ ] **"I like X" = the pattern, not the scope** — When someone references an existing feature, they mean the interaction model, not "clone it."

## During building

- [ ] **Mobile first** — Design and test for mobile before desktop. Not as an afterthought, as the default.
- [ ] **Can you see it?** — If you can't visually verify a UI change, say so. Don't declare it done.
- [ ] **Diagnose before redesigning** — When something looks wrong, ask what specifically is broken before changing the approach.
- [ ] **Check against the running system before acting** — Reproduce a finding live, confirm a tool fits the real environment, verify the actual value — *before* acting on it. Never act on an assumption or an unverified report. (Enforces "Don't assume, always check" + "Verify findings before acting".)
- [ ] **Shared-infra blast radius** — When a change alters how many requests a page makes, adds same-origin assets, or shifts load, check it against shared limits before deploying — nginx rate-limit zones, cgroup CPU/memory caps. Correct in isolation can still break a neighbour.

## After building

- [ ] **Verify end-to-end as the real user/role** — Exercise the actual flow as the unprivileged service user and in a real client (browser/device) — not as root, not unit tests alone. Root masks permission bugs; tests miss integration. If you built the same capability across several services, diff the instances (secret/file ownership, service user, cron user, paths) — divergence is a bug unless deliberate. (Enforces "Consistency is a feature".)
- [ ] **Test the logged-out path** — For any auth-gated change, load it with no session (incognito/real browser) and keep browser-fetched assets public (manifest, service worker, fonts, icons). (Enforces "Cookieless logged-out surface".)
- [ ] **Private by default** — New GitHub repos are created private. Going public is an explicit, deliberate decision — never the default.
- [ ] **Reflection** — What went wrong? What would have caught it earlier? If there's a reusable principle, add it to Janmam.
- [ ] **Sync notes** — If this session surfaced a new pattern or principle, log it in the Product sync folder.

## Wrapping a sync

- [ ] **Clarify before closing** — Any question raised during the sync must be answered before the sync ends, unless explicitly deferred ("we'll clarify that later"). Don't carry silent unknowns out of a sync.
- [ ] **To-do review** — Read back every to-do from today's sync. Mark done items. Carry forward open items. Don't close the sync with unacknowledged open points.
- [ ] **Remind** — Before signing off, surface any open to-do's or unresolved questions to the user.
- [ ] **No duplicate context** — If the daily log already captured the session, sync notes only add *new patterns or principles*. No echo entries.

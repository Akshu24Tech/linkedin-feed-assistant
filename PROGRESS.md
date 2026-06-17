# Progress log — LinkedIn Feed Assistant

A running record of what we're building and why. Newest entries on top. The
big arc right now is the **packaging gate**: the work that has to be true before
this is worth shipping as a Claude Code plugin (see "The packaging gate" below).

---

## 2026-06-17 — First-run guard + `/feed verify`

**Done:**

- **`/feed verify` (gate task T1-#2).** New read-only command
  (`.claude/commands/feed/verify.md`). Opens each watchlist profile, reads the
  displayed name + headline, classifies every row OK / MISMATCH / DEAD /
  UNREACHABLE, and proposes URL corrections with approval before writing. Wired
  into the router + menu in `feed.md`. Built to catch the failure from the first
  scan: 4 watchlist rows pointed at the wrong person or a dead URL.
- **First-run guard (gate task T2-#6).** Added to `_shared.md`, enforced at the
  top of `scan.md`. If `watchlist.csv` still contains ≥3 of the 5 shipped
  example profiles AND `.feed-personalized` is absent, `scan` hard-stops and
  points the user at `/feed setup`. Stops a fresh installer from scanning the
  author's profiles.
- **`.feed-personalized` marker.** Git-ignored (see `.gitignore`). Present =
  current user has claimed this copy, so the guard stands down. Created for the
  owner's copy today since these 5 profiles are the owner's real watchlist.
  Documented in `CLAUDE.md`.

**Not done / known gaps:**

- `/feed verify` has **not** been run against live LinkedIn yet — needs the
  Claude in Chrome extension (Claude Desktop → Code tab). First live test is
  pending.
- The guard works, but nothing *writes* `.feed-personalized` automatically yet.
  That belongs to T2-#5 (idempotent `/feed setup`). Until then the marker is
  created by hand.

---

## The packaging gate (do not package until all pass)

### Tier 1 — Reliability (prove it survives use)

- [ ] #1 Run `/feed scan` on ≥10 separate days
- [x] #2 Add `/feed verify` to validate watchlist URLs (built 2026-06-17; live test pending)
- [ ] #3 Observe both token gates (freshness + relevance) firing live
- [ ] #4 Prove incident-handling paths (CAPTCHA halt, mis-click recovery) work

### Tier 2 — Decoupling (safe for someone who isn't you)

- [ ] #5 Make `/feed setup` idempotent and state-free (also: write `.feed-personalized`)
- [x] #6 Add first-run guard against seed data (built 2026-06-17)
- [ ] #7 Separate user state from shipped logic (ship empty-template csv/interests)
- [ ] #8 Audit repo for credential/state leakage (grep, not memory)

**Exit test:** a person who is not the author can clone this, run `/feed setup`,
and get a correct first scan against *their own* watchlist — without the author
in the room, and without it ever touching the author's data.

---

## How to update this file

When we finish or change something, add a dated entry at the top (under the
date heading) with **Done** and **Not done / known gaps**, and tick the matching
box in the packaging gate checklist. Keep it honest — record what's untested.

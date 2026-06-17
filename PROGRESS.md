# Progress log — LinkedIn Feed Assistant

A running record of what we're building and why. Newest entries on top. The
big arc right now is the **packaging gate**: the work that has to be true before
this is worth shipping as a Claude Code plugin (see "The packaging gate" below).

---

## 2026-06-17 — Idempotent `/feed setup` that arms the guard

**Done:**

- **`/feed setup` rewritten (gate task T2-#5).** Now idempotent and it writes
  the `.feed-personalized` marker — the missing half of the first-run guard.
  - Step 0 reads personalization state (claimed vs fresh/unclaimed) and branches.
  - Step 4 detects the shipped seed fingerprint in `watchlist.csv` and makes the
    user clear it before adding their own profiles. A fresh install can't keep
    the author's list.
  - Step 5 only writes `.feed-personalized` when claiming is honest: the user's
    own interests profile AND a non-empty, non-seed watchlist. Otherwise it
    blocks and says exactly what's missing.
  - Re-runs never clobber real content or delete the marker.
- Net effect: the guard is now self-arming. A new user runs `/feed setup`,
  personalizes, gets claimed; until then `/feed scan` stays blocked.

**Not done / known gaps:**

- Still untested end to end — no clean-clone dry run yet to confirm a fresh
  install is genuinely blocked until setup completes (that's the T2-#5 exit
  check and overlaps the #7 / clean-clone work).
- `watchlist.csv` / `interests.md` are still shipped with the owner's real data,
  not empty templates — that's T2-#7, still open.

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

- [x] #5 Make `/feed setup` idempotent and state-free; writes `.feed-personalized` (built 2026-06-17; clean-clone test pending)
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

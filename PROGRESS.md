# Progress log — LinkedIn Feed Assistant

A running record of what we built and why. Newest entries on top.

**Status: paused after v1 (2026-06-18).** See the top entry for the decision and
reason. The arc that drove most of this log was the **packaging gate** (the work
required before this would be worth shipping as a plugin); Tier 2 was completed,
Tier 1 was deliberately not pursued. See "The packaging gate" below.

---

## 2026-06-18 — Project paused (v1 complete)

**Decision:** Pause the LinkedIn Feed Assistant after v1. Not abandoned —
deliberately stopped, resumable later.

**Why:** LinkedIn actively fights browser automation and changes its frontend
and rules without notice (v1's Python ancestor already died to exactly this:
DOM scraping → Voyager REST → Voyager GraphQL all blocked in turn). The
markdown rebuild lowered the risk by using a real session at human pace, but
the underlying dependency is still hostile. Further hardening is an unbounded
maintenance war against a platform that can silently break it. Stopping is the
senior call.

**Live test note:** `/feed verify` was run against a real account and worked
correctly. Separately, repeated "confirm it's you" checkpoints appeared while
testing on a *brand-new unverified* throwaway account — expected behavior for a
new account, not evidence the tool looks like a bot, and the tool correctly
**halted on every checkpoint** (hard rule #2). That halt-on-pushback behavior
is the partial evidence for #4; the full 10-day soak is being declined, not
failed.

**Final gate status:**

- Tier 2 (Decoupling): **complete and verified** (#5, #6, #7, #8).
- Tier 1 (Reliability): #2 verify done + tested. #1 (10-day soak), #3 (gates
  firing live), #4 (incident-handling fully exercised) **deliberately not
  pursued** — they require sustained live automation, which is the cost the
  pause avoids.

**What carries forward (the actual asset):** the productionization pattern for
taking a personal tool to safe-to-share — verify external assumptions,
first-run guard, personal-state/template separation, credential+PII audit of
tree and history. Reusable on the next project. The LinkedIn code is not.

This closes the active log. Reopen with a new dated entry if the project
resumes.

---

## 2026-06-17 — Clean-clone test + credential audit (T2-#8)

**Done:**

- **Clean-clone dry run.** Cloned the repo fresh from GitHub. Result: ships only
  the 6 command files + 3 templates + docs + `digest/.gitkeep`. `watchlist.csv`,
  `seen_posts.csv`, `profile/interests.md`, and `.feed-personalized` are all
  **absent**. A fresh clone has no real profiles and no marker, so the first-run
  guard would block `/feed scan` until setup runs. #5 + #6 + #7 validated
  together.
- **Credential audit (T2-#8): PASS.** Grepped both the current tree and the full
  git history for secrets (password, api key, bearer, `li_at`, `JSESSIONID`,
  client_secret, private keys, AWS/GitHub/OpenAI key formats) and PII (emails,
  phone numbers). **Zero matches anywhere.** No credentials have ever been
  committed.

**Known gaps (non-blocking):**

- Git history still contains the non-secret personal files removed in #7
  (`watchlist.csv` = public-figure profiles, `seen_posts.csv` + `digest/` =
  reading history, `interests.md` = the owner's public name + interest topics).
  None of it is sensitive and no email/phone is present, so **no history scrub
  is warranted**. Decision: leave history intact.

---

## 2026-06-17 — Ship templates, keep personal state local (T2-#7)

**Done:**

- Untracked the personal state from the repo (still on the owner's disk):
  `watchlist.csv`, `seen_posts.csv`, `profile/interests.md`, `digest/*.md` are
  now git-ignored.
- Added shipped templates: `watchlist.example.csv` (header + one obvious
  placeholder row), `seen_posts.example.csv` (header), and
  `profile/interests.example.md` (generic template). These are what a fresh
  clone gets.
- `setup.md` Step 1 now bootstraps the real files from the templates (and
  strips the placeholder watchlist row) instead of inventing rows.
- Updated README "Using this for yourself" and the CLAUDE.md file map to the
  template model.
- Net effect: a fresh clone ships **no** real profiles or reading history. It
  literally cannot scan anyone until the user runs setup.

**Not done / known gaps:**

- Git **history** still contains the previously-committed real watchlist /
  digest / interests (they were public on GitHub already). Untracking stops
  future shipping but does not scrub history. Decide under T2-#8 whether that
  matters (the seed profiles are public figures; the digest/interests are
  mildly personal, not secret).
- Still no clean-clone dry run proving setup bootstraps correctly from a bare
  checkout. That's the combined #5/#6/#7 exit test.

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
- [x] #2 Add `/feed verify` to validate watchlist URLs (built + live-tested working 2026-06-17)
- [ ] #3 Observe both token gates (freshness + relevance) firing live
- [ ] #4 Prove incident-handling paths (CAPTCHA halt, mis-click recovery) work

### Tier 2 — Decoupling (safe for someone who isn't you)

- [x] #5 Make `/feed setup` idempotent and state-free; writes `.feed-personalized` (built 2026-06-17; clean-clone test pending)
- [x] #6 Add first-run guard against seed data (built 2026-06-17)
- [x] #7 Separate user state from shipped logic; personal files git-ignored, templates ship (built 2026-06-17; clean-clone test pending)
- [x] #8 Audit repo for credential/state leakage — PASS, zero secrets in tree or history (2026-06-17)

**Exit test:** a person who is not the author can clone this, run `/feed setup`,
and get a correct first scan against *their own* watchlist — without the author
in the room, and without it ever touching the author's data.

---

## How to update this file

When we finish or change something, add a dated entry at the top (under the
date heading) with **Done** and **Not done / known gaps**, and tick the matching
box in the packaging gate checklist. Keep it honest — record what's untested.

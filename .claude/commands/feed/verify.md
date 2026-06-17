# /feed verify — validate every watchlist profile resolves correctly

Read `_shared.md` first. Usage: `/feed verify` (optionally `/feed verify "Name"`
to check one row).

Purpose: catch the failure that poisoned the first scan — a watchlist row whose
vanity points at the **wrong person** (a `harrison-chase` who is a Wiz sales
director, an `andrewng` who curates coffee) or at a **dead/404** profile. This
is a health check that runs *before* you trust a scan, not part of scanning.

**Read-only.** This flow opens profile pages and reads the name + headline. It
never saves, follows, or writes anything on LinkedIn. The only files it may
write are `watchlist.csv` (corrected URLs), and only after the user approves
each fix.

## Step 0 — Verify environment

Run Step 0 from `_shared.md` (Chrome + LinkedIn check).

## Step 1 — Build the check list

Read `watchlist.csv`. Check **every** row regardless of Status (a paused or
active row can both be wrong). With `/feed verify "Name"`, check just that row.

Show the rows you are about to verify as a short table (Name, Vanity, Note).

## Step 2 — Per profile: resolve and compare

For each row, one at a time, human pace:

1. Navigate to the row's `Profile URL`
   (`https://www.linkedin.com/in/<vanity>/`).
2. Read the **displayed name** and the **headline** on the profile page.
3. Classify the row into one of:
   - **OK** — displayed name matches the watchlist `Name` (allow normal
     variation: middle names, "Dr.", ordering) AND the headline is consistent
     with the `Note`. No action.
   - **MISMATCH** — the page loads but shows a different person (name and/or
     headline clearly do not match the row's Name/Note).
   - **DEAD** — 404, "profile not found", "this page doesn't exist", or a
     redirect to a generic page.
   - **UNREACHABLE** — auth wall, "sign in to view", or the page won't load.
     Not necessarily wrong; flag for a manual look.

If LinkedIn pushes back (CAPTCHA, unusual-activity banner, rate-limit wall),
**halt immediately** per the hard rules: report which rows were checked and
their results so far, and let the user decide when to resume.

## Step 3 — For each MISMATCH or DEAD: find the correct profile

Do not auto-edit. For each flagged row:

1. Try to find the right profile for the row's `Name` (+ `Note` as a hint):
   WebSearch `site:linkedin.com/in "<Name>"`, or LinkedIn search via Chrome.
2. Open the candidate, read its name + headline, and confirm it fits the Note.
3. **Show the user the old vanity, the proposed new URL, and the headline you
   found, and get an explicit yes before changing anything.** Vanity-name
   collisions are exactly how the bad rows got in; never swap silently.

If no confident match is found, leave the row unchanged and recommend the user
either fix it by hand or `/feed` remove it.

## Step 4 — Apply approved corrections

For each fix the user approved: update that row's `Vanity` and `Profile URL` in
`watchlist.csv`. Preserve every other column and every untouched row exactly
(per `CLAUDE.md` rule 5). Do not touch `Last Checked`, `Times Checked`, or
`Avg Score` — verification is not a scan.

## Step 5 — Report

End with a results table:

```
| Name | Result | Action |
|---|---|---|
| Harrison Chase | OK | none |
| Yohei Nakajima | MISMATCH | corrected vanity → yoheinakajima |
| Denny Zhou | DEAD | corrected URL → denny-zhou-7695487 |
| Andrej Karpathy | UNREACHABLE | flagged for manual check |
```

Then state plainly whether the watchlist is now clean (every row OK or
corrected) or still has rows needing a human decision. A scan is only
trustworthy once verify comes back clean.

## Step 6 — Commit

If the folder is a git repo and any row changed, make one commit covering
`watchlist.csv`. Message: `verify YYYY-MM-DD: N rows corrected`. Do not push.
If nothing changed, do not commit.

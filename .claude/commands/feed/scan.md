# /feed scan — the core flow

Read `_shared.md` first. Arguments:

- `--force "Name"` — scan only that profile, bypassing both gates.
- `--threshold N` — save threshold for this run (default 7).

## Step 0 — Verify

Run Step 0 from `_shared.md` (Chrome + LinkedIn check). Read
`profile/interests.md`; if missing/template, stop and run `/feed setup`.

## Step 1 — Build the scan queue

Read `watchlist.csv`. Take `active` rows, then apply the gates from
`CLAUDE.md`:

- Skip if `Last Checked` < 20 hours ago → count as "fresh, skipped".
- If `Times Checked >= 5` and `Avg Score < 3.0` → set Status to `paused`,
  count as "paused for low relevance" (report it; the user can reactivate).

With `--force "Name"`, the queue is just that profile, no gates.

If the queue is empty, say so with the reason per profile and stop — that's a
successful run, not an error.

Show the queue as a short table before starting.

## Step 2 — Per profile: extract

For each profile in the queue, one at a time:

1. Navigate to `https://www.linkedin.com/in/<vanity>/recent-activity/all/`.
2. Extract post cards per the navigation rules in `_shared.md` (14-day window,
   max 5 cards, stop after 2 processed).
3. Dedup each post URL against `seen_posts.csv` (normalized). Already-seen
   posts are skipped without analysis.

If LinkedIn pushes back (CAPTCHA, warning banner, rate-limit interstitial):
**stop the whole scan now**, update `watchlist.csv` for profiles already
completed, write the digest for what was processed, and report exactly where
it halted.

## Step 3 — Per post: score

For each NEW post, score 1–10 against `profile/interests.md` and produce:

- **Score** (1–10) and **content type** (e.g. tool_announcement, deep_dive,
  hot_take, tutorial, hiring, hype)
- **Summary** — 2–3 sentences, what the post actually says
- **Key insight** — the one concrete takeaway
- **Comment draft** — only for posts at/above threshold; style per
  `_shared.md` (first person, no em-dashes, no filler)
- **Content angle** — only at/above threshold; how this could seed the user's
  own post

## Step 4 — Save qualifying posts in LinkedIn

For each post with score >= threshold: on the post card, click `⋯` (top-right
of the card) → **Save**. Confirm the "Saved" toast/state. This is the ONLY
permitted write action on LinkedIn. No per-post user confirmation is needed
for saving (it's private and reversible); everything else stays read-only.

If the Save click fails (menu changed, item missing), record `Saved=no` and
continue — never improvise another interaction to compensate.

## Step 5 — Update state

- Append every examined NEW post to `seen_posts.csv` (saved or not).
- Update the profile's row in `watchlist.csv`: `Last Checked`,
  `Times Checked` +1, recompute `Avg Score`.
- Preserve all untouched rows exactly.

## Step 6 — Write the digest

Append to (or create) `digest/YYYY-MM-DD.md`:

```markdown
# Feed digest — YYYY-MM-DD

## Scan at HH:MM (threshold N)

### <Author> — score 9 — tool_announcement
<post URL>

**Summary:** ...
**Key insight:** ...
**Comment draft:**
> ...
**Content angle:** ...

### ...next saved post...

## Skipped this scan
| Author | Score | Why |
|---|---|---|
```

Only posts that were saved get full sections; below-threshold posts get one
table row each.

## Step 7 — Commit

If the folder is a git repository, make one commit covering the run's state
changes (`watchlist.csv`, `seen_posts.csv`, the digest file). Message format:
`scan YYYY-MM-DD: N posts examined, M saved`. Do not push — the user pushes
when they want to.

## Step 8 — Report

End with a summary table: profiles scanned / fresh-skipped / paused, posts
examined / already seen / saved, and a reminder that saved posts are in
LinkedIn under **My Items → Saved posts**
(`https://www.linkedin.com/my-items/saved-posts/`) with the analysis in
today's digest file.

# Shared context — loaded by all /feed sub-flows

Working directory: the repo root (where `watchlist.csv` lives).
Today's date: read from system (YYYY-MM-DD).

Re-read `CLAUDE.md` hard rules before any flow that touches LinkedIn. The two
that matter most: **the only LinkedIn write action allowed is the post "Save"
button**, and **halt on any CAPTCHA / unusual-activity warning**.

---

## Step 0 — Chrome + LinkedIn verification (every flow that opens LinkedIn)

1. Confirm the Claude in Chrome extension is connected. If not, stop and tell
   the user: open Chrome, make sure the Claude extension is installed and the
   browser tool is available, then rerun.
2. Open `https://www.linkedin.com/feed/` in the existing Chrome window.
3. If redirected to a login page, stop: the user must sign in manually first.
4. Confirm the account: open the "Me" menu or check the nav avatar name. If
   `profile/interests.md` has a `## Me` section with the user's name, verify it
   matches. If something's off, say exactly what to fix; never type credentials.

---

## First-run guard (every flow that scans or saves)

Before scanning, make sure this is the user's own watchlist and not the example
data the project ships with. A fresh installer must never scan the author's
seeded profiles.

**The seed fingerprint** — the example watchlist shipped with the project:

```
harrison-chase-961287118, yoheinakajima, andrewyng,
denny-zhou-7695487, andrej-karpathy-9a650716
```

**The personalization marker** — a file `.feed-personalized` in the repo root.
It is git-ignored, so it never ships. Its presence means "the current user has
claimed this copy as their own." `/feed setup` writes it once the user confirms
their identity and watchlist.

**Guard procedure:**

1. If `.feed-personalized` exists → this copy is claimed. Skip the guard,
   proceed.
2. Else, read `watchlist.csv`. If **3 or more** of the seed-fingerprint vanities
   are present → this is almost certainly the unmodified example data.
   **Hard-stop.** Do not scan. Tell the user:
   > This looks like the project's example watchlist, not yours. Run
   > `/feed setup` to build your own interest profile and watchlist before
   > scanning, so you don't scan the author's profiles.
3. Else (no marker, but the seed data has been replaced) → allow the flow.
   `/feed setup` will still formalize personalization; this guard's only job is
   to block the shipped example data.

This guard applies to `scan`. It does **not** apply to `verify` — verifying the
example data is harmless and read-only.

---

## Interest profile — `profile/interests.md`

The single source of truth for scoring. Free-form prose/bullets:
interest areas, what counts as high-signal, what to always skip, the user's
voice notes for comment drafts. If the file is missing or still the template,
stop and run the interview from `setup.md` Step 3 instead of guessing.

---

## watchlist.csv columns

```
Name, Vanity, Profile URL, Note, Status, Added, Last Checked, Times Checked, Avg Score
```

- `Vanity` — the slug in `linkedin.com/in/<vanity>/`. Derive it from the
  Profile URL.
- `Status` — `active` or `paused`. Only `active` rows are scanned.
- `Avg Score` — running mean of all post scores for this profile, one decimal.
  Update with: `new_avg = (old_avg * old_count + sum(new_scores)) / (old_count + len(new_scores))`
  where counts are numbers of scored posts. Track scored-post count implicitly:
  if simpler, recompute from `seen_posts.csv` rows for that author.
- `Last Checked` — `YYYY-MM-DD HH:MM`, set every time the profile is scanned
  (even if 0 new posts).

## seen_posts.csv columns

```
Post URL, Author, Posted Date, Scanned Date, Score, Saved
```

- `Post URL` is the dedup key. Normalize: strip query params and trailing
  slashes before comparing or appending.
- `Saved` — `yes` if the LinkedIn Save click succeeded, else `no`.
- A post already present in this file is NEVER re-analyzed. Skip silently and
  count it in the summary as "already seen".

## digest/ files

One file per scan day: `digest/YYYY-MM-DD.md`. If the file already exists
(second scan same day), append a new `## Scan at HH:MM` section, never
overwrite. Format is defined in `scan.md` Step 6.

---

## Activity page navigation

A profile's posts live at:

```
https://www.linkedin.com/in/<vanity>/recent-activity/all/
```

Read the visible post cards from the page. For each card capture: author (the
watchlist person, or note if it's a repost), post text (click "…see more" /
"…more" inside the card if truncated), relative age ("3d", "1w"), and the post
URL (from the card's timestamp link or `⋯ → Copy link to post`; prefer the
timestamp link, it avoids a menu click). Convert relative ages to absolute
dates using today's date; "1w" = 7 days, "1mo" = 31 days (fails the 14-day
window).

Post-window rules from `CLAUDE.md` apply: 14-day window, max 5 cards, stop
after 2 processed recent posts, unknown age passes.

If the page shows no posts (private activity, empty page), record the check in
`watchlist.csv` and move on; mention it in the summary.

---

## Writing style for comment drafts

First person, the user's voice, 1–3 sentences, adds a concrete point or asks a
sharp question. No em-dashes. No "Great post!" filler, no emojis unless the
user's voice notes say otherwise, no hashtags. The draft should read like a
busy practitioner typed it.

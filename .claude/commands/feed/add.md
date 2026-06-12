# /feed add — add a profile to the watchlist

Read `_shared.md` first. Usage: `/feed add <profile-url-or-name> [note]`

## Step 1 — Resolve the profile

- If given a URL: extract the vanity slug from `linkedin.com/in/<vanity>/`.
- If given only a name: use WebSearch (`site:linkedin.com/in "<name>"`) or
  LinkedIn search via Chrome to find the profile. **Show the user the URL and
  headline you found and get a yes before adding** — vanity-name collisions
  are common and adding the wrong person silently poisons the watchlist.

## Step 2 — Dedup

If the vanity already exists in `watchlist.csv`: if `paused`, offer to
reactivate; otherwise say it's already tracked and stop.

## Step 3 — Append

Add the row: Name, Vanity, Profile URL, Note (or empty), Status `active`,
Added = today, Last Checked empty, Times Checked 0, Avg Score empty. Preserve
all other rows exactly.

## Step 4 — Confirm

Show the updated watchlist as a markdown table. If the user seems to be adding
several, offer to take the rest as a batch list.

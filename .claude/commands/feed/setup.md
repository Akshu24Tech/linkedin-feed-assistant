# /feed setup — first-run check

Read `_shared.md` first. Goal: by the end, the user can run `/feed scan` and it
just works.

## Step 1 — Workspace check

Confirm `watchlist.csv`, `seen_posts.csv`, `profile/interests.md`, and
`digest/` exist. Recreate any missing file from the formats in `_shared.md`
(headers only, no invented rows).

## Step 2 — Chrome + LinkedIn

Run Step 0 from `_shared.md`. Report clearly: extension connected? signed in?
as whom?

## Step 3 — Interest profile interview

If `profile/interests.md` is missing, empty, or still the placeholder
template, build it WITH the user, conversationally. Don't assign homework.
Ask, in one message:

- What topics/technologies do you want surfaced? (their high-signal list)
- What should always be skipped, even if popular?
- Anything about your voice for comment drafts? (tone, things you'd never say)
- Your name as it appears on LinkedIn (for the account check in Step 0).

Then write `profile/interests.md` with sections: `## Me` (name),
`## High signal`, `## Always skip`, `## Voice notes`. If the file already has
real content, read it, show a short summary, and ask if anything changed.

## Step 4 — Seed the watchlist

If `watchlist.csv` has no data rows, ask the user for 3–5 profiles to start
with (names or URLs) and add them via the `add.md` procedure. A small
watchlist scanned often beats a huge one scanned never.

## Step 5 — Wrap up

Show: profiles tracked (markdown table), interests summary (one line), and
tell the user the daily habit is just `/feed scan`.

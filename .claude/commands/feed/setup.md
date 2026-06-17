# /feed setup — first-run check + claim this copy

Read `_shared.md` first. Goal: by the end, this copy is personalized to the
user, the `.feed-personalized` marker is written, and `/feed scan` will run.

Setup is **idempotent** — safe to re-run any time. It never clobbers real user
content; it reads what exists, shows it back, and only changes what the user
confirms.

## Step 0 — Read personalization state

Decide which mode you're in before doing anything:

- **Already claimed** — `.feed-personalized` exists. This is a re-run. Skip the
  hard gating below; just refresh whatever the user wants (interests, watchlist)
  and re-confirm. Do not delete the marker.
- **Fresh / unclaimed** — `.feed-personalized` is absent. The first-run guard
  is armed. Your job this run is to get the user to a state where claiming the
  copy is honest, then write the marker.

## Step 1 — Workspace check (bootstrap from templates)

The personal state files are git-ignored and do not ship — a fresh clone only
has the `*.example.*` templates. Bootstrap the real files if missing:

- `watchlist.csv` ← copy `watchlist.example.csv`, then **delete the placeholder
  example row** (keep the header).
- `seen_posts.csv` ← copy `seen_posts.example.csv` (header only).
- `profile/interests.md` ← copy `profile/interests.example.md` (this counts as
  "still the template" in Step 3 until the user fills it in).
- `digest/` already exists (`digest/.gitkeep`); leave it empty.

Never invent watchlist rows during bootstrap. A bootstrapped watchlist is
empty (header only), which the first-run guard treats as unclaimed until the
user adds their own profiles in Step 4.

## Step 2 — Chrome + LinkedIn

Run Step 0 from `_shared.md`. Report clearly: extension connected? signed in?
as whom?

## Step 3 — Interest profile interview

Build `profile/interests.md` WITH the user, conversationally. Don't assign
homework. Ask, in one message:

- What topics/technologies do you want surfaced? (their high-signal list)
- What should always be skipped, even if popular?
- Anything about your voice for comment drafts? (tone, things you'd never say)
- Your name as it appears on LinkedIn (for the account check in Step 0).

Then write `profile/interests.md` with sections: `## Me` (name),
`## High signal`, `## Always skip`, `## Voice notes`.

Treat the file as **not yet personalized** if it is missing, empty, still the
placeholder template, or still carries the "Seeded from the previous Python
agent" note with an unconfirmed `## Me` name. In the fresh/unclaimed mode you
must complete this step — do not skip to claiming with a template profile.

If the file already has real, user-confirmed content, read it, show a short
summary, and ask if anything changed.

## Step 4 — Seed the watchlist (and clear shipped example data)

Check `watchlist.csv` against the **seed fingerprint** in `_shared.md`.

- **If 3+ of the shipped example profiles are still present** (the unmodified
  example list): tell the user these are the project's examples, not theirs.
  Offer to clear them. Do not let a fresh install keep the author's profiles.
- Then ask for 3–5 profiles of their own (names or URLs) and add them via the
  `add.md` procedure. A small watchlist scanned often beats a huge one scanned
  never.

A copy is only "personalized" when the watchlist is **non-empty and is the
user's own** (not the unmodified seed fingerprint).

## Step 5 — Claim the copy

Decide whether claiming is honest:

- **Claim allowed** when BOTH are true: `profile/interests.md` is the user's own
  (Step 3), AND `watchlist.csv` is non-empty and not the unmodified seed
  fingerprint (Step 4).
- **Claim blocked** otherwise. Tell the user exactly what is still missing
  (e.g. "your interests profile is still the template" or "the watchlist is
  still the example profiles") and stop before writing the marker. Do not write
  `.feed-personalized` on a half-set-up copy — that would disarm the guard
  prematurely.

When claiming is allowed and the marker does not already exist, write
`.feed-personalized` with:

```
<Name> — claimed YYYY-MM-DD.
This watchlist is the owner's own curated list, not example data.
Git-ignored on purpose: a fresh clone must NOT inherit this marker.
```

(`.feed-personalized` is git-ignored, so it never ships. See `.gitignore`.)

## Step 6 — Wrap up

Show: profiles tracked (markdown table), interests summary (one line), and
whether the copy is now claimed. If claimed, tell the user the daily habit is
`/feed verify` once to confirm the watchlist is clean, then `/feed scan`. If
claiming was blocked, restate the one thing to finish and re-run `/feed setup`.

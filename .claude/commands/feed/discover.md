# /feed discover — find new high-signal profiles to track

Read `_shared.md` first. Usage: `/feed discover [topic or hint] [--limit N]`

Goal: surface people worth adding to the watchlist, with evidence, and add
only the ones the user approves. This flow is **read-only research** — it never
writes to LinkedIn and never edits `watchlist.csv` until the user says yes.

## Step 1 — Load the target

Read `profile/interests.md`. The `## High signal` list is what "good" means;
the `## Always skip` list is a negative filter on candidates too (e.g. skip
people who are mostly motivational/hype accounts). If the file is missing or
still the template, stop and run `/feed setup` first — discovering against an
empty interest profile just returns noise.

If the user passed a topic/hint (`/feed discover RAG eval`), focus the search
on it. With no argument, search across the whole `## High signal` list.

Default candidate count: 5. `--limit N` overrides it.

## Step 2 — Search the web

Use `firecrawl_search` as the primary tool (per the project's web provider);
fall back to `WebSearch` if it is unavailable. Run a few focused queries, not
one broad one. Good query shapes:

- `site:linkedin.com/in <high-signal topic>`
- `<topic> researcher OR founder OR engineer linkedin`
- `who to follow <topic> linkedin 2026`
- authors of well-known tools/papers in the high-signal areas

Prefer people who **post regularly** about the high-signal topics, not just
famous names with dormant feeds — the watchlist is scanned for *recent
activity*, so a quiet profile is low value even if the person is prominent.

After a `firecrawl_search`, call `firecrawl_search_feedback` with the search ID
(refunds a credit and improves quality).

## Step 3 — Resolve and dedup

For each candidate, capture: Name, the `linkedin.com/in/<vanity>` URL, a
one-line headline/role, and the **evidence** (what they post about, a recent
example or the source that surfaced them).

Drop a candidate if:

- its vanity already exists in `watchlist.csv` (active or paused) — say so,
  don't re-propose it;
- you can't find a real `linkedin.com/in/` URL (no guessing vanities);
- it matches the `## Always skip` profile (hype/motivational-only).

Do **not** open LinkedIn in Chrome here. Identity verification happens at
add-time in `add.md`, which already shows the URL + headline and waits for a
yes. Keeping discovery web-only means it works without an active LinkedIn
session and adds no automation surface.

## Step 4 — Present candidates for approval

Show a ranked table — best fit first:

```
| # | Name | Headline | Why it fits (evidence) | Profile URL |
|---|------|----------|------------------------|-------------|
```

Then ask which to add: "all", a list of numbers, or none. **Add nothing until
the user picks.** This is the approve-to-add gate; it is the whole point of the
command.

## Step 5 — Add the approved ones

For each approved candidate, follow `.claude/commands/feed/add.md` (Step 1
onward) — including its URL/headline confirm and its dedup check. Carry a short
`Note` derived from the evidence (e.g. "posts on RAG eval + retrieval bench").
If the user said "all", you may batch them, but still honor `add.md`'s
per-profile identity confirm for any candidate whose URL you are not certain of.

## Step 6 — Report

Show the updated watchlist as a markdown table, how many were added vs.
skipped (and why skipped — duplicate, declined, unverifiable), and remind the
user the next `/feed scan` will pick up the new profiles. Discovery writes no
digest and saves nothing on LinkedIn.

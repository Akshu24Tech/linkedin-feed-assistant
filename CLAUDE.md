# CLAUDE.md — LinkedIn Feed Assistant

Project rules loaded automatically when Claude works in this folder.

## What this is

A Claude Code feed-intelligence workspace. It monitors a watchlist of LinkedIn
profiles, scores their recent posts against the user's interest profile, saves
qualifying posts natively in LinkedIn (the post's "Save" option), and writes a
local digest with the analysis LinkedIn can't store (score, key insight,
comment draft, content angle).

There is **no automation code** in this repo. All LinkedIn interaction happens
through the official **Claude in Chrome** extension driving the user's real,
logged-in browser session, human-paced, while the user watches. This is what
keeps the workflow inside LinkedIn's rules — never reintroduce Playwright,
headless browsers, stored cookies, or credential files.

The intended user path: run `/feed setup` once, then `/feed scan` whenever you
want a sweep of your watchlist.

## Where things live

| File | Purpose |
|---|---|
| `CLAUDE.md` (this file) | Project-wide hard rules, canonical values |
| `README.md` | Human-readable overview + setup |
| `.claude/commands/feed/_shared.md` | Shared rules loaded by every `/feed` sub-flow |
| `.claude/commands/feed/{setup,scan,add}.md` | Per-flow procedures |
| `profile/interests.md` | The user's interest profile — what to score posts against |
| `watchlist.csv` | Tracked profiles + per-profile stats. Source of truth for WHO to scan |
| `seen_posts.csv` | Every post ever examined. Source of truth for dedup |
| `digest/YYYY-MM-DD.md` | One file per scan day: analysis of posts that were saved |

## Hard rules

1. **READ-ONLY on LinkedIn, except the "Save" button.** The only write action
   ever performed on LinkedIn is clicking a post's `⋯ → Save`. Never post
   comments, never react, never follow/connect, never send messages. Comment
   drafts in the digest are drafts the user pastes manually if they choose.
2. **Stop on pushback.** If LinkedIn shows a CAPTCHA, an "unusual activity"
   warning, or a verification wall mid-scan, halt immediately, report which
   profiles were completed, and let the user decide when to resume.
3. **Human pace.** Process one profile at a time, one tab. Never open profiles
   in parallel. If the watchlist is long, scanning fewer profiles well beats
   racing through all of them.
4. **No credentials anywhere.** Never store LinkedIn passwords, cookies,
   session tokens, or API keys in this repo. Login state lives in the user's
   Chrome profile only.
5. **`watchlist.csv` and `seen_posts.csv` are sources of truth.** When
   updating, preserve every untouched row exactly.
6. **Date format:** always `YYYY-MM-DD` (timestamps `YYYY-MM-DD HH:MM`).
7. **Render CSV contents as clean markdown tables** in chat — never raw CSV.
8. **No em-dashes (—) in comment drafts or any text the user might paste to
   LinkedIn.** Use commas, periods, or parentheses. Em-dashes read as
   AI-written. Internal files (digest headers, notes) are fine.
9. **Comment drafts are first person, in the user's voice** (builder, direct,
   technical, no hype). Never draft in third person.

## Token-efficiency gates (mirror of the old Python agent)

Applied during `/feed scan`, computed from `watchlist.csv`:

- **Freshness gate:** skip a profile if `Last Checked` is less than **20 hours**
  ago.
- **Relevance gate:** skip a profile if `Times Checked >= 5` AND
  `Avg Score < 3.0`. Mark it `paused` in the Status column and tell the user.
- **Bypass:** `/feed scan --force "Name"` rescans one profile regardless of
  gates. The user can also flip Status back to `active` by hand.

## Post-window rules

- Only consider posts from the last **14 days**.
- Examine at most **5 post cards** per profile; stop early once **2** recent
  posts have been processed. Don't scroll endlessly.
- If a post's age can't be determined from the page, let it through (avoid
  false negatives).

## Scoring

Score each new post **1–10** against `profile/interests.md`. Threshold to save:
**7** (overridable per run: `/feed scan --threshold 6`).

- 8–10: clearly within interest areas, concrete/technical
- 5–7: related but shallow or partially relevant
- 1–4: off-topic, hype, motivational, job posts, engagement bait

## Out of scope for v1 (do not build ahead)

Profile discovery, post generation from content angles, scheduling, comment
posting. These are later iterations. Do not start them while v1 is being
refined.

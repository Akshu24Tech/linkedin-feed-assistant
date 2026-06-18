# LinkedIn Feed Assistant

> **Status: v1 complete and working. Paused on purpose.**
> The agent does what it set out to do. Further work is deliberately not
> pursued because LinkedIn actively fights browser automation and changes its
> rules without notice, which makes ongoing hardening a losing maintenance war.
> See [Why this is paused](#why-this-is-paused) and `PROGRESS.md`. Resumable
> any time if that calculus changes.

An AI agent that watches the LinkedIn profiles you care about, scores their
posts against your interests, **saves the good ones natively in LinkedIn**
(My Items → Saved posts), and writes a local digest with what LinkedIn can't
store: the score, the key takeaway, a ready-to-paste comment draft in your
voice, and a content angle for your own posts.

Built with **zero lines of automation code**. The entire agent is markdown
instruction files executed by Claude Code, driving your real browser through
the official Claude in Chrome extension.

## Why this exists

Scrolling LinkedIn is passive and inefficient. The problem that started this
project, in the words of the original project notes:

> While scrolling LinkedIn, I missed many useful posts and the chance to be
> part of the conversation or capture useful information in time.

Concretely:

- Good posts get buried — no way to filter the feed by what actually matters to you
- "See more" truncation means you miss context even on posts you do see
- You miss the comment window — by the time you see a post, it's 3 days old
  and the conversation is over
- No system captures insights for later use: content ideas, reference
  material, things worth building on

The original sketch of the fix, drawn before any code was written:

```
LinkedIn post (content + comment)
        ↓
  what is it about? does it matter? how can it be used?
        ↓
  score it against my interests
        ↓
  score ≥ 7 → save to LinkedIn Saved Posts
        ↓
  keep the summary + insight + comment draft
```

This repo is that sketch, finally working end to end.

## The journey — two versions, one hard lesson

**v1** (`linkedin_agent`, Python) took the obvious engineering route: Playwright
browser automation with saved cookies, Gemini for relevance scoring, SQLite
memory so no post is ever analyzed twice, Notion for storage, a FastAPI
server, and eventually a custom Chrome extension. Seventeen Python files.
Genuinely good architecture.

Then LinkedIn blocked it. Three extraction approaches died in a row:

1. **DOM scraping** — sidebar garbage, ads, posts truncated at 200 chars
2. **Voyager REST API** — 400 errors, deprecated
3. **Voyager GraphQL API** — the queryId rotates with every LinkedIn frontend
   deploy; unmaintainable by design

The first fix was realizing the logged-in browser already on the machine was
the one client LinkedIn trusts. The second fix, inspired by
[claude-linkedin-assistant](https://github.com/FarzamHejaziK/claude-linkedin-assistant)
(a markdown-only Claude Code job-search assistant), was more radical: delete
the code entirely.

| v1 (Python agent) | v2 (this repo) |
|---|---|
| Playwright + cookies / custom extension + FastAPI | Official **Claude in Chrome** extension, real logged-in session |
| Gemini API scoring (`analyzer.py`) | Claude scores directly from `profile/interests.md` |
| SQLite memory + token gates (`memory.py`) | `watchlist.csv` + `seen_posts.csv`; gates as written rules |
| Notion storage (`notion_saver.py`) | Native LinkedIn Save + `digest/*.md` |
| Scheduler, retry, logger, setup_check | Not needed |

Nothing of value was lost in the rewrite. The token-efficiency gates (skip
profiles checked in the last 20 hours, pause consistently irrelevant ones),
the 14-day post window, the URL dedup, the 1–10 scoring rubric — all survived
as written rules instead of code.

The trade-off accepted on purpose: scans run when you trigger them in Claude
Code with Chrome open. No unattended cron scraping — that constraint is the
feature that keeps it inside LinkedIn's rules. There is no bot to detect,
because there is no bot.

## Requirements

- **Claude Desktop app** (the Code tab) or Claude Code CLI
- **Claude in Chrome extension**, installed in a Chrome where you're signed
  into LinkedIn

## Usage

Open this folder in Claude Code (Desktop app → Code tab → new session in this
folder), then:

```
/feed setup                      one-time: verify Chrome + LinkedIn, build your
                                 interest profile, seed the watchlist
/feed scan                       the daily habit: scan, save, digest
/feed scan --force "Name"        rescan one profile, bypassing the skip gates
/feed scan --threshold 6         lower the save bar for one run
/feed add <profile-url> [note]   track a new profile
/feed discover [topic]           web-search for new high-signal profiles, approve-to-add
```

Pause or remove profiles by editing `watchlist.csv` (Status column).

Saved posts land in LinkedIn at
<https://www.linkedin.com/my-items/saved-posts/>; the matching analysis is in
`digest/YYYY-MM-DD.md`.

The daily loop in practice: morning coffee → `/feed scan` → open Saved posts
to read what mattered → grab a comment draft from the digest if a conversation
is worth joining.

## Safety model

- The **only** write action on LinkedIn is the post "Save" button. No
  comments, reactions, connects, or DMs are ever sent. Comment drafts are
  text you paste yourself.
- One profile at a time, human pace, your real browser session, while you
  watch.
- Any CAPTCHA or warning halts the scan immediately.
- No credentials, cookies, or API keys stored anywhere in this repo.

## Using this for yourself

Fork or clone, then run `/feed setup` — it does the work for you:

1. It bootstraps your personal state from the shipped templates
   (`watchlist.example.csv`, `seen_posts.example.csv`,
   `profile/interests.example.md`).
2. It interviews you to build `profile/interests.md` in your voice.
3. It has you add your own profiles, then claims the copy as yours.

The personal state files (`watchlist.csv`, `seen_posts.csv`,
`profile/interests.md`, `digest/*.md`) are git-ignored, so they never ship and
your reading history stays on your machine. Only the templates are tracked. A
fresh clone is incapable of scanning anyone else's profiles until you set it up.

## What shipped (done)

- [x] v1 — Python agent: Playwright → blocked → custom extension + FastAPI (see Journey above)
- [x] v2 — markdown rebuild: scan watchlist, native save, daily digest
- [x] `/feed discover` — find new high-signal profiles via web search, approve-to-add
- [x] `/feed verify` — check every tracked profile resolves to the right person (tested working)
- [x] Productionization pass — first-run guard, idempotent setup, personal state
      separated from shipped templates, repo + git history audited clean of secrets

## Deliberately out of scope (paused, not abandoned)

These were scoped and consciously declined, not forgotten. Each would require
sustained live automation against LinkedIn, which is exactly the cost the pause
decision avoids.

- `/feed post` — turn saved content angles into post drafts in your voice
- golden-dataset scoring checks — a fixed set of posts with expected scores
- 10-day reliability soak — prove the gates and incident-handling over real use
- package as an installable Claude Code plugin

## Why this is paused

The agent works. Stopping here is a decision, not a failure.

LinkedIn actively fights browser automation and changes its frontend and rules
without notice. v1 already survived that once: the original Python agent died
when DOM scraping, the Voyager REST API, and the Voyager GraphQL API were each
blocked in turn. The markdown rebuild dodged that by using a real logged-in
session at human pace, but the underlying truth did not change. Building on top
of a platform that can silently break you is renting, not owning. Every hour
spent hardening this can be erased by one deploy on their side. That is an
unbounded maintenance war against a hostile dependency, and the senior move is
to not fight it.

So v1 ships, it works, and it is paused on purpose.

## What I'd carry forward

The LinkedIn code was never the real point. The transferable skill is the part
that survives this project: how you take a tool from "works on my machine" to
"safe for a stranger to run." That pattern, proven here, is reusable on any
future project:

- A **verify** step that validates external assumptions before trusting them
- A **first-run guard** so a fresh install can never act on the author's data
- **Personal state separated from shipped code** (git-ignored data, shipped templates)
- A **credential/PII audit** of the working tree and full git history

That is what I keep. See `PROGRESS.md` for the full build log and the gate that
defined "done enough to share."

---

*Built by Akshu Grewal | [GitHub](https://github.com/Akshu24Tech) | The previous
Python iteration lives on as the archaeology layer of this README.*

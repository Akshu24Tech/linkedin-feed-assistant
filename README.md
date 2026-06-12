# LinkedIn Feed Assistant

A Claude Code workspace that monitors LinkedIn profiles you care about, scores
their posts against your interests, **saves the good ones natively in LinkedIn**
(My Items → Saved posts), and writes a local digest with the analysis LinkedIn
can't store: score, key insight, a ready-to-paste comment draft, and a content
angle for your own posts.

## History — why this exists

This is the second incarnation of the project. The first
(`E:\Projects\linkedin_agent`) was a Python agent: Playwright scraping with
saved cookies (Mode B), later a custom Chrome extension + FastAPI bridge
(Mode A), Gemini for scoring, SQLite for memory, Notion for storage. It worked,
but Mode B was exactly the headless-browser-with-stored-credentials pattern
LinkedIn's bot policy targets, and the whole stack (server, extension,
retry/rate-limit code, API keys) existed to work around that.

Inspired by [claude-linkedin-assistant](https://github.com/FarzamHejaziK/claude-linkedin-assistant)
(a markdown-only Claude Code job-search assistant), this rebuild deletes the
automation layer entirely:

| v1 (Python agent) | v2 (this repo) |
|---|---|
| Playwright + cookies / custom extension + FastAPI | Official **Claude in Chrome** extension, real logged-in session |
| Gemini API scoring (`analyzer.py`) | Claude scores directly from `profile/interests.md` |
| SQLite memory + token gates (`memory.py`) | `watchlist.csv` + `seen_posts.csv`; gates as written rules |
| Notion storage (`notion_saver.py`) | Native LinkedIn Save + `digest/*.md` |
| Scheduler, retry, logger, setup_check | Not needed |

The trade-off accepted on purpose: scans run when you trigger them in Claude
Code with Chrome open. No unattended cron scraping — that constraint is the
feature that keeps it policy-safe.

## Requirements

- **Claude Code desktop app** (not the web version — it needs local files and
  Chrome control)
- **Claude in Chrome extension**, installed in a Chrome where you're signed
  into LinkedIn

## Usage

Open this folder in Claude Code desktop, then:

```
/feed setup                      one-time: verify Chrome + LinkedIn, build your
                                 interest profile, seed the watchlist
/feed scan                       the daily habit: scan, save, digest
/feed scan --force "Name"        rescan one profile, bypassing the skip gates
/feed scan --threshold 6         lower the save bar for one run
/feed add <profile-url> [note]   track a new profile
```

Pause or remove profiles by editing `watchlist.csv` (Status column).

Saved posts land in LinkedIn at
<https://www.linkedin.com/my-items/saved-posts/>; the matching analysis is in
`digest/YYYY-MM-DD.md`.

## Safety model

- The **only** write action on LinkedIn is the post "Save" button. No
  comments, reactions, connects, or DMs are ever sent. Comment drafts are
  text you paste yourself.
- One profile at a time, human pace, your real browser session.
- Any CAPTCHA or warning halts the scan immediately.
- No credentials, cookies, or API keys stored anywhere in this repo.

## Roadmap (one iteration at a time)

- [x] v1 — scan watchlist, native save, daily digest
- [ ] v2 — `/feed discover`: find new high-signal profiles via WebSearch, approve-to-add
- [ ] v3 — `/feed post`: turn saved content angles into post drafts in your voice
- [ ] later — golden-dataset scoring checks (a fixed set of posts with expected
      scores, re-run after any change to `interests.md` or `scan.md`)

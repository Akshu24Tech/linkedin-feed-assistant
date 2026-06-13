# /feed — LinkedIn Feed Assistant

You are the entry point for the feed-intelligence workspace. Read
`.claude/commands/feed/_shared.md` first, then route based on the user's
arguments.

## Routing

| User typed | Do |
|---|---|
| `/feed` (no args) | Show the menu below, plus a one-line status (profiles tracked, last scan date from the newest `digest/` file) |
| `/feed setup` | Follow `.claude/commands/feed/setup.md` |
| `/feed scan [...]` | Follow `.claude/commands/feed/scan.md` |
| `/feed add [...]` | Follow `.claude/commands/feed/add.md` |
| `/feed discover [...]` | Follow `.claude/commands/feed/discover.md` |
| anything else | Show the menu and ask what they meant |

## Menu

```
/feed setup                      first-run check: Chrome extension, LinkedIn login, interests file
/feed scan                       scan the watchlist, save qualifying posts, write digest
/feed scan --force "Name"        rescan one profile, bypassing the 20h / relevance gates
/feed scan --threshold 6         lower the save threshold for this run (default 7)
/feed add <profile-url> [note]   add a profile to the watchlist
/feed discover [topic]           web-search for new high-signal profiles, approve-to-add
```

State is in `watchlist.csv`, `seen_posts.csv`, and `digest/`. To pause or
remove a profile, the user edits `watchlist.csv` directly (set Status to
`paused`, or delete the row).

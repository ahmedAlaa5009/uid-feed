# uid-feed

Auto-synced pool of numeric channel UIDs for the video backend (branch: `feed`).

## Files

### `uids_feed.json`
```json
{"uids": ["1661347828146896898", "1661719653385367553", "..."]}
```
- Full current pool: deduped, sorted ascending by uid **value**.
- `uid` is a snowflake ID — a larger value = newer account
  (creation time = `(uid >> 22) + 1288816000000` milliseconds).
- Sorted by value, **not** by discovery time — use `uids_recent.json` for
  "what was pushed recently".

### `uids_recent.json` — what was recently pushed
```json
{"updated": "2026-10-17T02:14:00Z",
 "total": 2935,
 "last_added_at": "2026-10-17T02:07:17Z",
 "recent_count": 1959,
 "recent": [{"u": "1786...", "t": "2026-10-17T02:07:17Z"}, ...],
 "removed_count": 43,
 "removed": []}
```
- `recent` — **newest-first** log of **every uid in the feed**, each with its push
  timestamp (UTC). Backfilled from the full push history.
- `removed` — uids dropped after they were verified dead.
- **Select the latest N**: take the first N entries — `recent[:50]`, `recent[:100]`, ...
- **Select by time**: filter on `t` (everything pushed after a given timestamp).

Latest 50 pushed:
```bash
curl -s "https://raw.githubusercontent.com/ahmedAlaa5009/uid-feed/feed/uids_recent.json" | python3 -c "import json,sys;d=json.load(sys.stdin);print('\n'.join(e['u'] for e in d['recent'][:50]))"
```

Everything pushed since 2026-10-17 00:00 UTC:
```bash
curl -s "https://raw.githubusercontent.com/ahmedAlaa5009/uid-feed/feed/uids_recent.json" | python3 -c "import json,sys;d=json.load(sys.stdin);print('\n'.join(e['u'] for e in d['recent'] if e['t']>='2026-10-17T00:00:00Z'))"
```

Both files are rewritten continuously by the harvester fleet.

## If you are an agent adding uids

Publish **only** via the campaign helper — it updates BOTH `uids_feed.json` AND
`uids_recent.json` (recent additions with timestamps + removals):

```bash
cd /mnt/e/JS/.temp/harvest_state && python3 -c "import thumb_harvest; thumb_harvest.push_feed()"
```

Never hand-edit or hand-push these JSON files; never bypass the recent file.
Coverage repair tool: `python3 sync_recent.py`.
Full agent rules: `/mnt/e/JS/.temp/harvest_state/AGENTS.md`.

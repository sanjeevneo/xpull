# xpull

Pull tweets, threads, and articles from X/Twitter without login or API keys (advanced features).

```bash
node scripts/fx-fetch.mjs "https://x.com/naval/status/1002103360646823936"
```

Zero dependencies. Node 18+. JSON to stdout.

---

## Install

```bash
git clone https://github.com/sanjeevneo/xpull.git
```

No `npm install`. Nothing to build.

## Usage

### Single tweet

```bash
node scripts/fx-fetch.mjs "https://x.com/elonmusk/status/1585341984679469056"
```

### Thread (OP only)

Walks the self-reply chain upward. Same author only.

```bash
node scripts/fx-fetch.mjs "https://x.com/naval/status/1002103497725173760" --thread
```

### Full thread from root (Grok)

FxTwitter walks threads backward (child → parent). For forward traversal from a root tweet, Grok handles it in one call.

```bash
export XAI_API_KEY="xai-..."
node scripts/grok-x-search.mjs thread "https://x.com/naval/status/1002103360646823936"
```

### Replies

```bash
node scripts/grok-x-search.mjs replies "https://x.com/elonmusk/status/1585341984679469056"
```

### Search

```bash
node scripts/grok-x-search.mjs search "transformer architecture"
```

```bash
node scripts/grok-x-search.mjs search "AI safety" --from elonmusk
```

### Media understanding (optional)

Add `--images` or `--video` to any Grok command to analyse media in posts. Off by default — adds token cost.

```bash
node scripts/grok-x-search.mjs search "infographic" --images
```

## Output

Raw [FxTwitter](https://github.com/FxEmbed/FxEmbed) tweet objects as a JSON array:

```json
[
  {
    "text": "Entering Twitter HQ – let that sink in!",
    "author": { "name": "Elon Musk", "screen_name": "elonmusk" },
    "created_at": "Wed Oct 26 18:45:58 +0000 2022",
    "likes": 1245542,
    "retweets": 158453,
    "views": 120000000,
    "media": { "videos": [{ "url": "...", "thumbnail_url": "..." }] },
    "replying_to_status": null
  }
]
```

X Articles include full text via `article.content.blocks[]`. Threads return a chronological array.

## How it works

**fx-fetch.mjs** calls the FxTwitter public API — the same backend that powers tweet embeds in Discord and Telegram. Free, unauthenticated, structured JSON. Thread reconstruction follows `replying_to_status` upward through the self-reply chain until the author changes or the root is reached.

**grok-x-search.mjs** uses xAI's `x_search` tool on the [Responses API](https://docs.x.ai). Server-side query against X's backend — not a scraper, not prompt-based guessing. Handles forward thread traversal, replies, and search.

## Grok (optional)

Requires `XAI_API_KEY` — set in environment or `.env` file. Get one at [console.x.ai](https://console.x.ai).

| | |
|---|---|
| Model | `grok-4-1-fast-non-reasoning` |
| x_search cost | $5 / 1,000 calls |
| Token cost | $0.20 / $0.50 per M tokens |
| Daily cap | 20 (override: `GROK_DAILY_CAP` env var) |

State tracked in `.grok-state.json`. Resets daily.

## OpenClaw

Available on [ClawHub](https://clawhub.ai/sanjeevneo/xpull):

```bash
clawhub install xpull
```

Or manually — copy into your workspace skills folder and restart the gateway:

```bash
cp -r xpull ~/.openclaw/workspace/skills/
```

## Limitations

- FxTwitter is a third-party public API. No SLA, no uptime guarantee.
- Thread walking only goes upward (child → parent). Root → children requires Grok.
- Public tweets only. Protected accounts return 401.
- Some older tweets return 404 — X restricts guest API access to older content.
- Grok features require a paid xAI API key.

## Licence

[MIT](LICENSE)

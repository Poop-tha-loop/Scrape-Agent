# scrapepack-agent

A Claude Code plugin that drives [scrapepack](https://github.com/YOUR-USERNAME/scrapepack) — social-platform scraping across YouTube, X, Instagram, and TikTok — with quota and spend checks before anything live.

The plugin ships **the agent, not the scraper.** You still need a `scrapepack` checkout on the machine where you run it.

## Install

```
/plugin marketplace add YOUR-USERNAME/scrapepack-agent
```

```
/plugin install scrapepack@scrapepack-agent
```

Then point it at your scrapepack checkout:

```bash
export SCRAPEPACK_HOME=~/path/to/scrapepack
```

## Use

Prompt it in plain language — "find outlier videos about ai agents", "crawl this channel", "what would a full run cost?" — or invoke the skill directly:

```
/scrape outliers -q "ai agents"
```

For multi-account runs or anything needing judgment about the results, ask for the `scrapepack-scraper` agent by name.

## What's in it

| Component | Purpose |
|---|---|
| `scrapepack-scraper` agent | Full cost model, scoring semantics, failure playbook. Use for real work. |
| `/scrape` skill | Lightweight entry point — locate, selftest, check cost, run, report. |

## Why it checks cost first

Two platforms spend real resources, and neither failure mode is obvious until it bites:

- **YouTube** has a hard 10,000 unit/day cap. `search.list` costs 100 units per call, so keyword discovery is ~100× more expensive than crawling a channel. A cold 4-keyword run is ~500–600 units — about 17 runs before you're locked out for the day.
- **X** charges per post read. A multi-account run is a spending decision.

The agent runs `quota` and `cost` before any live scrape and reports the numbers rather than proceeding silently.

## Known sharp edge

The fixture backend — which replays recorded data with no keys and no network — covers the **actor platforms** (X, Instagram, TikTok) but **not** the YouTube CLI commands. `outliers`, `channel`, and `transcript` construct `YouTubeAPI` directly and hard-require `YOUTUBE_API_KEY` at startup regardless of backend setting, failing with:

```
ValueError: YouTube API key required.
```

Setting `SCRAPEPACK_YOUTUBE_BACKEND=fixture` does not help. The agent knows this and will point you at a real key rather than sending you round the backend-variable loop.

## Credentials

`YOUTUBE_API_KEY`, `X_BEARER_TOKEN`, `IG_ACCESS_TOKEN` + `IG_USER_ID`, `TIKTOK_CLIENT_KEY` + `TIKTOK_CLIENT_SECRET`, `APIFY_TOKEN`, `SCRAPEPACK_API_KEY`.

Set these in your own shell. The agent will not write them to a file, and this repo contains none.

## Limits it won't work around

TikTok's Research API is academic-only and its Display API is own-account only. Instagram needs Business/Creator on both ends and returns no video view counts. Transcripts use an endpoint outside ToS that's blocked from datacenter IPs — it works from a laptop, not a cloud runner. These are policy walls, not missing features.

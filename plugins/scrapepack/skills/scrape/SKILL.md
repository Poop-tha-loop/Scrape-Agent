---
name: scrape
description: Run a social-platform scrape through scrapepack. Use when the user asks to scrape, pull, or fetch data from YouTube, X/Twitter, Instagram, or TikTok; to find outlier or breakout videos; to crawl a channel; to fetch a transcript; to check scraping quota; or to estimate what a research run will cost. Also triggers on "/scrape".
version: 0.1.0
argument-hint: [what to scrape, e.g. outliers -q "ai agents" | channel UC... | transcript VIDEO_ID]
---

# Run a scrape

## 1. Locate scrapepack

The plugin ships the agent, not the scraper. Find the `scrapepack` checkout before running anything:

```bash
if [ -n "$SCRAPEPACK_HOME" ] && [ -d "$SCRAPEPACK_HOME/scrapepack" ]; then
  echo "$SCRAPEPACK_HOME"
else
  find ~ -maxdepth 4 -type d -name scrapepack -not -path '*/node_modules/*' 2>/dev/null | head -5
fi
```

Every command below runs from that directory. If nothing turns up, tell the user to clone scrapepack and set `SCRAPEPACK_HOME` — do not try to install it yourself.

## 2. Confirm the install is healthy

```bash
python3 -m scrapepack.cli selftest
```

This is fully offline and needs no keys. If it fails, stop and report which check failed — running a scrape against a broken install wastes quota and money.

## 3. Decide live vs. fixture

**Default to fixtures.** Only go live when the user explicitly asks for real, current data.

Fixtures need no keys and no network, and cover the actor platforms (X, Instagram, TikTok). YouTube commands always need a real `YOUTUBE_API_KEY` — the fixture default does not cover them.

## 4. Check the bill before spending

For anything live:

```bash
python3 -m scrapepack.cli quota
python3 -m scrapepack.cli cost --repo PATH
```

Report remaining quota and dollar cost **to the user before running.** If the run costs more than a few dollars or would blow the daily cap, stop and ask. Never spend on your own judgment.

## 5. Run it

```bash
python3 -m scrapepack.cli outliers -q "QUERY" --size 20 --format json
python3 -m scrapepack.cli channel CHANNEL_ID --format json
python3 -m scrapepack.cli transcript VIDEO_ID
python3 -m scrapepack.cli run ACTOR --input '{"...": "..."}'
```

Use `--format json` when you need to rank or filter; `text` when the user just wants to read it.

For anything involving multiple accounts, judgment about which results matter, or interpreting scores against a brief, hand off to the **`scrapepack-scraper`** agent instead of running commands inline — it carries the full cost model, the scoring semantics, and the failure playbook.

## 6. Report

In this order: what ran, what it cost, top results by `multiplier`, then any `_metricsPartial` fields or `FAILED_ACCOUNTS`. Numbers first.

Never write credentials into a file, and never set `SCRAPEPACK_ALLOW_UNSANCTIONED`.

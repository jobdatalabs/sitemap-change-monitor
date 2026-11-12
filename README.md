# Sitemap Change Monitor: New, Updated & Removed Pages

[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-00a76f?style=for-the-badge)](https://apify.com/jobdata_labs/sitemap-change-monitor)

> **Hosted version:** run it in the cloud, no install, pay per result: **[Sitemap Change Monitor: New, Updated & Removed Pages](https://apify.com/jobdata_labs/sitemap-change-monitor)**

Extract every URL from a website's XML sitemaps and find out **which pages were added, updated or removed since the last check**. Point it at a competitor, a client site or your own site, schedule it, and get a clean change list instead of a full crawl.

## What it does

- **Finds sitemaps automatically**: give it a website and it reads the `Sitemap:` lines in robots.txt, falling back to `/sitemap.xml`. Or give it a sitemap URL directly.
- **Follows sitemap indexes and `.xml.gz` files**, so large sites with nested sitemaps work out of the box.
- **Change tracking**: with "Only changes since last run" on, each run returns only pages that are `new`, `updated` (newer `lastmod`) or `removed`.
- **Filters**: keep only `/blog/` or `/product/` URLs, exclude tag pages, or ignore pages not modified since a date.
- **Lightweight and polite**: reads only sitemap files, never the pages themselves. Identifies itself with its own user agent and respects robots.txt.

## Typical uses

- **Competitor content tracking**: see every new blog post, landing page or product a competitor publishes.
- **SEO monitoring**: catch pages that silently disappear from your sitemap, or a sudden wave of new URLs.
- **Site migrations**: compare the URL inventory before and after a relaunch.
- **Content pipelines**: feed only new or updated URLs into a scraper, an LLM or a RAG index instead of re-crawling everything.
- **Client reporting**: weekly list of what changed on a client's site.

## How change tracking works

1. The first run with a given **State name** stores a snapshot of all URLs and their `lastmod` dates and returns all of them (`changeType: null`).
2. Each later run compares the sitemap with the snapshot and returns only:
   - `new`: the URL was not in the previous snapshot,
   - `updated`: its `lastmod` is different from the previous one,
   - `removed`: it was in the previous snapshot and is gone now.
3. The snapshot is then replaced with the current state.

`removed` is only reported when the whole sitemap was read without errors, limits or filters, so a partial read never produces false removals.

Schedule the Actor from the **Schedules** tab (daily is typical) and connect an integration (Slack, email, Google Sheets, webhook) to receive the changes.

## Input

| Field | Type | Description |
| :--- | :--- | :--- |
| `startUrls` | Array | Websites (`example.com`) or sitemap URLs. Required. |
| `onlyChanges` | Boolean | Return only new, updated and removed pages. Default `false`. |
| `stateKey` | String | Name of the stored snapshot. Default `default`. |
| `includePatterns` | Array | Keep only URLs containing one of these fragments. |
| `excludePatterns` | Array | Drop URLs containing any of these fragments. |
| `lastmodAfter` | String | Keep only URLs modified on or after this date (`2026-01-31`). |
| `maxUrlsPerSite` | Integer | Stop after this many URLs per site. `0` = no limit. Default `5000`. |
| `respectRobotsTxt` | Boolean | Skip sitemap files disallowed by robots.txt. Default `true`. |

### Example: weekly competitor blog tracker

```json
{
  "startUrls": ["https://competitor.example"],
  "onlyChanges": true,
  "stateKey": "competitor-blog",
  "includePatterns": ["/blog/"]
}
```

## Output

One dataset row per URL:

```json
{
  "site": "https://crawlee.dev",
  "url": "https://crawlee.dev/blog/crawlee-v3-16",
  "lastmod": "2026-11-05T00:00:00.000Z",
  "changefreq": "weekly",
  "priority": 0.5,
  "sitemapUrl": "https://crawlee.dev/sitemap.xml",
  "changeType": "updated",
  "previousLastmod": "2026-10-01T00:00:00.000Z",
  "checkedAt": "2026-11-12T14:34:03.058Z"
}
```

A per-site summary (sitemap files read, URLs found and saved, counts of new/updated/removed, errors) is stored in the key-value store under `SUMMARY` and printed in the log.

## Pricing

You pay per URL saved to the dataset; the current rate is on the pricing tab. With change tracking on, unchanged pages are not saved and not charged, so a scheduled monitor on a stable site costs very little. Use `maxUrlsPerSite` to cap the cost of a run.

## FAQ

**The site has no sitemap. What happens?**
The run fails with a clear message. This Actor only reads sitemaps; it does not crawl pages.

**Why are some pages never marked as updated?**
Updates are detected from the sitemap's `lastmod` field. If a site does not publish `lastmod`, only new and removed pages can be detected.

**A sitemap shows "Disallowed by robots.txt".**
The website asks automated clients not to fetch that file, and this Actor respects that by default.

**Can I track several sites in one run?**
Yes. Each site has its own snapshot inside the same state name.

---

## Run it via API

```bash
curl -X POST "https://api.apify.com/v2/acts/jobdata_labs~sitemap-change-monitor/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"startUrls": ["https://crawlee.dev"], "maxUrlsPerSite": 100}'
```

```js
import { ApifyClient } from 'apify-client';
const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor('jobdata_labs/sitemap-change-monitor').call({
    "startUrls": [
        "https://crawlee.dev"
    ],
    "maxUrlsPerSite": 100
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
```

## Use it from AI agents (MCP)

```json
{
  "mcpServers": {
    "sitemap-change-monitor": {
      "url": "https://mcp.apify.com?tools=jobdata_labs/sitemap-change-monitor",
      "headers": { "Authorization": "Bearer <YOUR_APIFY_TOKEN>" }
    }
  }
}
```

## More tools by jobdata_labs

All tools: [apify.com/jobdata_labs](https://apify.com/jobdata_labs) · [SEO Audit Crawler](https://apify.com/jobdata_labs/seo-audit-crawler) · [Sitemap Change Monitor](https://apify.com/jobdata_labs/sitemap-change-monitor) · [RSS Feed Monitor](https://apify.com/jobdata_labs/rss-feed-monitor) · [Tech Stack & DNS Detector](https://apify.com/jobdata_labs/tech-stack-dns-detector)

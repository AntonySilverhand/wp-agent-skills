---
name: wp-health-check
description: Run a health check on the WordPress site — performance, broken links, security basics, plugin updates, and accessibility. Use before a site launch, after major changes, when something seems broken, or when the user asks to "check the site", "audit the site", or "make sure everything is working". Reports issues with severity levels and recommended fixes.
license: MIT
compatibility: Requires WP-CLI and WP REST API. Full scan requires curl. Performance metrics are approximations, not Lighthouse scores.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-health-check

Full site health audit. Reports issues by severity with recommended fixes.

## Arguments

- `DEPTH` — `quick` (home page only, ~30s) or `full` (all pages, ~5 min). Default: `quick`.

## Subagent strategy

Run all five checks in parallel using subagents — they are fully independent:
- **Agent 1**: WordPress/plugin update status (WP-CLI)
- **Agent 2**: Performance (curl timing)
- **Agent 3**: Broken link scan
- **Agent 4**: Security checks
- **Agent 5**: Accessibility audit

Collect all results, then compile the prioritised report. This turns a 5-step sequential
audit into a single parallel sweep — typically 5× faster.

## Step 1 — WordPress updates

```bash
wp core check-update
wp plugin list --update=available --format=json | jq '[.[].name]'
wp theme list --update=available --format=json | jq '[.[].name]'
```

Flag security updates as **CRITICAL**. Feature updates as **LOW**.

## Step 2 — Performance check

```bash
# Measure TTFB and total load
curl -s -o /dev/null -w "TTFB: %{time_starttransfer}s | Total: %{time_total}s | Size: %{size_download}B\n" \
  "$WP_SITE_URL"
```

Thresholds:
- TTFB > 500ms → **HIGH** (caching not active or slow server)
- Page size > 3MB → **MEDIUM** (unoptimized images or too many scripts)
- TTFB < 200ms + size < 1MB → note as healthy

Check for caching plugin:
```bash
wp plugin list --status=active | grep -E "litespeed|w3-total-cache|wp-super-cache|autoptimize"
```
No caching plugin on a live site → **HIGH**.

## Step 3 — Broken link scan

**Quick**: home page only. **Full**: all published pages.

```bash
# Extract all hrefs from a page
curl -s "$WP_SITE_URL" | grep -oP 'href="[^"]*"' | grep -oP '(?<=href=")[^"]*'
```

For internal links: check HTTP status. Expect 200 or 301.
For external links: HEAD request, flag non-200.
404 on internal link → **HIGH**. 404 on external → **MEDIUM**.

## Step 4 — Security basics

```bash
wp config get WP_DEBUG                  # should be false on production
wp config get DISALLOW_FILE_EDIT        # should be true
curl -s -o /dev/null -w "%{http_code}" "$WP_SITE_URL/wp-config.php"  # expect 403
curl -s -o /dev/null -w "%{http_code}" "$WP_SITE_URL/?author=1"       # expect 404 or 301
```

- `WP_DEBUG` true in production → **HIGH**
- `wp-config.php` accessible → **CRITICAL**
- User enumeration enabled → **MEDIUM**

## Step 5 — Accessibility basics (quick: home page, full: all pages)

For each checked page:
```bash
curl -s "$URL" | grep -c 'alt=""'           # images with empty alt
curl -s "$URL" | grep -c '<img '            # total images
curl -s "$URL" | grep -c '<main'            # should be >= 1
curl -s "$URL" | grep -oP '<h[1-6]' | sort # check heading hierarchy
```

- Images without non-empty alt text → **MEDIUM** per image
- No `<main>` landmark → **MEDIUM**
- Heading hierarchy skips (h1 → h3) → **LOW**

## Gotchas

- **`curl` TTFB includes DNS lookup.** Run twice and use the second result — DNS is cached on the second call.
- **WP-CLI `check-update` pings WordPress.org.** It will fail on servers with no outbound internet. Catch the error and note "update check unavailable".
- **`WP_DEBUG` may be set in a `.env` file, not `wp-config.php`**, in which case `wp config get WP_DEBUG` won't see it. If the site uses a `.env` loader, also check that file.
- **User enumeration check via `/?author=1`**: some themes redirect this to the author archive intentionally. Check the response body — if it's a profile page, enumeration is active. If it's a 404 page, it's blocked.

## Output

Prioritised issue list:

**CRITICAL** — fix immediately (security, data loss risk)
**HIGH** — fix before launch (performance, broken user flows)
**MEDIUM** — fix soon (SEO, accessibility, minor UX)
**LOW** — nice to have

Each issue: what it is, where it is, one-line fix.

End with: `"X critical, X high, X medium, X low issues found."`
Suggest `wp-seo-pass all` if SEO issues were found.

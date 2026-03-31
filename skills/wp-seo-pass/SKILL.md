---
name: wp-seo-pass
description: Run an SEO optimization pass on one page or the entire site — meta titles, descriptions, Open Graph tags, structured data, and image alt text. Use when the site is ready to launch, when the user asks to "improve SEO", "make the site searchable", "add meta descriptions", or after adding new pages. Also use when missing alt text is reported.
license: MIT
compatibility: Requires WordPress 6.4+, WP REST API. Works with or without Yoast/RankMath — falls back to theme head injection if no SEO plugin is found.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-seo-pass

SEO optimization for one page or all pages. Updates meta, Open Graph, structured data, alt text.

## Arguments

- `TARGET` — page title/ID, or `all` for every published page
- `FOCUS_KEYWORD` — (optional) primary keyword to optimize for

## Subagent strategy

When `TARGET` is `all`, use subagents for speed:
- Launch one subagent per page to audit and generate meta in parallel
- Collect all results, then apply updates sequentially (REST API is stateful)
- For a 10-page site this cuts processing time from ~10 minutes to ~2 minutes

For single-page runs, no subagents needed.

## Step 1 — Detect SEO plugin

```bash
wp plugin list --status=active --format=json | jq '[.[].name]'
```

- **Yoast SEO active**: update via `yoast_wpseo_title` and `yoast_wpseo_metadesc` post meta
- **RankMath active**: update via `rank_math_title` and `rank_math_description` post meta
- **Neither**: inject via `wp_head` hook in `functions.php`

## Step 2 — Audit each target page

Fetch page content and check:

| Check | Target |
|---|---|
| Meta title | Under 60 chars, not the default site title |
| Meta description | 120–160 chars, includes focus keyword if set |
| H1 count | Exactly 1 |
| Images missing alt | 0 |
| Internal links | At least 1 outgoing |

## Step 3 — Write meta

For each page:
- **Title**: `<Specific Page Topic> | <Site Owner Name>` — under 60 chars
- **Description**: 1–2 sentences. What the page contains + why someone would care. Under 160 chars. Include `FOCUS_KEYWORD` naturally if provided.

Don't use "Welcome to", "Home page of", or the site URL in titles.

## Step 4 — Add Person schema (home + about pages only)

Inject JSON-LD structured data:

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "<full name>",
  "url": "<WP_SITE_URL>",
  "jobTitle": "<if known>",
  "sameAs": ["<social URLs>"],
  "description": "<one sentence bio>"
}
```

Add via `wp_head` hook or Yoast's schema override if available.

## Step 5 — Fix missing alt text

For every image block with no alt text, generate alt text from:
1. The `IMAGE:` placeholder description (if it was set by `wp-new-page`)
2. The filename
3. The surrounding page context

Update each image block via REST API (full content body required).

## Step 6 — Open Graph tags

Ensure these exist on all pages (add to theme `<head>` if no plugin handles them):

```html
<meta property="og:title" content="<meta title>" />
<meta property="og:description" content="<meta description>" />
<meta property="og:image" content="<featured image URL>" />
<meta property="og:type" content="website" />
<meta name="twitter:card" content="summary_large_image" />
```

## Gotchas

- **Yoast stores meta in `_yoast_wpseo_title` and `_yoast_wpseo_metadesc` post meta**, not in standard fields. The REST API exposes these under `yoast_head_json` but writing requires the Yoast REST extension or direct post meta update.
- **The REST API `title` field updates the page H1, not the meta title.** These are separate. Always update meta title through the SEO plugin's fields.
- **JSON-LD `sameAs` requires full URLs** (https://...), not handles (@username).
- **Open Graph `og:image` must be an absolute URL**, not a relative path.

## Output

Per-page table:

| Check | Before | After |
|---|---|---|
| Meta title | — | "Photography | Lavon" (48 chars) |
| Meta description | missing | 138 chars ✓ |
| Images with alt | 2/5 | 5/5 ✓ |
| H1 count | 0 | 1 ✓ |

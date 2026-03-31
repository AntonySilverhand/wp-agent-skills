---
name: wp-update-section
description: Edit a specific section, block pattern, or piece of content on the site. Use when the user wants to change copy, swap a photo, add or remove a timeline item, update a quote, fix a heading, or change anything on a specific page or across a reusable pattern. Makes surgical edits — does not rebuild the whole page. Examples: "change the hero headline", "update the about section", "add a new project to the gallery", "fix the contact email".
license: MIT
compatibility: Requires WordPress 6.4+, WP REST API
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-update-section

Surgical edits to a specific section or page. Show a diff before applying.

## Arguments

- `TARGET` — pattern slug (edits the pattern file, affects all pages using it) **or** a page title/ID (edits that page only)
- `CHANGE` — plain English description of the edit

## Step 1 — Locate the target

**If target is a pattern slug:**
Read `patterns/<slug>.php` from the theme directory.
Note: changes here affect every page using this pattern.

**If target is a page title:**
```bash
curl -s "$WP_SITE_URL/wp-json/wp/v2/pages?search=<title>" \
  | jq '.[0] | {id, title: .title.rendered, link}'
```
Then fetch `content.raw` for the page.

## Step 2 — Parse the change

Identify the minimum change needed. Categories:
- **Text**: find the exact text node, replace content only
- **Image**: find `IMAGE:` placeholder or `wp:image` block, update `url` and `alt`
- **Add item** (e.g., timeline entry): find the containing block group, append a new item with matching markup
- **Remove item**: find and delete the specific block
- **Animation**: update `data-anim` attribute on the wrapper `<div>`

**Do not refactor surrounding blocks.** Touch only what's described.

## Step 3 — Show diff before applying

```
--- BEFORE
+++ AFTER
@@ relevant block @@
- <h2 class="wp-block-heading">...</h2>
+ <h2 class="wp-block-heading">...</h2>
```

Ask: "Apply this change?" — wait for confirmation before writing.

## Step 4 — Apply

**Pattern file:**
Write updated `patterns/<slug>.php`.
```bash
wp cache flush
```

**Page:**
```bash
curl -s -X POST "$WP_SITE_URL/wp-json/wp/v2/pages/<id>" \
  -H "Authorization: Basic $(echo -n "$WP_USER:$WP_APP_PASSWORD" | base64)" \
  -H "Content-Type: application/json" \
  -d '{"content": "<full updated content>"}'
```

When updating a page, always POST the full `content` field — the API replaces, doesn't merge.

## Step 5 — Verify

Fetch the updated resource and confirm the change is present:
```bash
curl -s "$WP_SITE_URL/wp-json/wp/v2/pages/<id>" | jq '.content.raw' | grep "<changed text>"
```

## Gotchas

- **The REST API requires the full content body on update**, not just the changed block. Fetch first, patch in memory, send the full string.
- **Pattern changes don't propagate to pages that have been individually edited** in the block editor — those pages store a copy of the pattern's content, not a reference. If a page drifts, you'll need to update it separately.
- **Block comment attributes use JSON** (`{"align":"wide"}`). A malformed JSON attribute silently breaks that block in the editor while still rendering on the frontend.
- **`wp cache flush` flushes object cache, not page cache.** If a caching plugin (LiteSpeed, WP Super Cache) is active, also clear its page cache from WP Admin.

## Output

Diff of what changed. Preview URL to verify visually.

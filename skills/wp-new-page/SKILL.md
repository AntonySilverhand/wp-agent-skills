---
name: wp-new-page
description: Create a new WordPress page by assembling existing block patterns, filling in real content, and publishing or saving as draft. Use when the user wants to add a new page to the site — home, about, portfolio, contact, projects, or any custom page. Takes a title and content brief; outputs a complete, ready-to-review page. Does not create generic Lorem Ipsum — all content is specific to the brief.
license: MIT
compatibility: Requires WordPress 6.4+, WP REST API, and at least one lavon-pattern registered. Run wp-create-pattern first if no patterns exist.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-new-page

Assemble a new page from existing block patterns and real content. Publish or save as draft.

## Arguments

- `PAGE_TITLE` — the page name (e.g., "About", "Portfolio", "Contact")
- `CONTENT_BRIEF` — what this page is about and what it should contain
- `STATUS` — `draft` (default) or `publish`

## Subagent strategy

Use two subagents in parallel before assembly:
- **Agent 1**: Inventory available patterns from the REST API
- **Agent 2**: Draft the content for each section based on the brief

Merge the results (which patterns exist + what content to put in them), then assemble.
This saves a full round-trip of reading → thinking → writing.

## Step 1 — Inventory available patterns

```bash
curl -s "$WP_SITE_URL/wp-json/wp/v2/block-patterns/patterns" \
  | jq '[.[] | select(.categories[] | contains("lavon-patterns")) | .slug]'
```

Only use patterns that exist. If a needed pattern is missing, run `wp-create-pattern` first.

## Step 2 — Choose the page structure

Pick a pattern sequence based on the page type. Default recipes:

| Page | Pattern sequence |
|---|---|
| Home | nav-sticky → hero-cinematic → about-split → gallery-masonry → quote-feature → contact-minimal |
| About | nav-sticky → hero-cinematic → about-split → timeline-scroll → quote-feature |
| Portfolio | nav-sticky → hero-cinematic → gallery-masonry → contact-minimal |
| Contact | nav-sticky → about-split → contact-minimal |

For custom pages: think narrative arc — hook → story → proof → connection.
Only include a pattern if the content brief gives you real material for it.

## Step 3 — Write real content

Replace every placeholder. No Lorem Ipsum — use the brief.

- **Headlines**: short, specific, striking. Not "Welcome to my site" — something that only fits this person.
- **Body text**: warm, first-person, genuine (unless the brief says otherwise).
- **CTAs**: specific ("See my photography" not "Learn more").
- **Image slots**: write `IMAGE:<descriptive caption>` as a placeholder. Example: `IMAGE:portrait outdoors in warm afternoon light`. These get resolved by `wp-add-media`.
- **Alt text**: write descriptive alt text for every image slot, even placeholders.

## Step 4 — Assemble block markup

Combine selected patterns in order. Substitute real content into each pattern's placeholders.

Wrap in template parts if the nav is a separate template part:
```
<!-- wp:template-part {"slug":"header","theme":"lavon"} /-->
```

## Step 5 — Create via REST API

```bash
curl -s -X POST "$WP_SITE_URL/wp-json/wp/v2/pages" \
  -H "Authorization: Basic $(echo -n "$WP_USER:$WP_APP_PASSWORD" | base64)" \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"$PAGE_TITLE\", \"content\": \"<markup>\", \"status\": \"$STATUS\"}"
```

## Gotchas

- **Block markup must use proper WordPress comment delimiters** (`<!-- wp:heading -->..<!-- /wp:heading -->`). Raw HTML without block comments renders fine on the frontend but is uneditable in the block editor.
- **`base64` encoding of credentials differs across platforms.** On macOS, use `base64` without `-w 0`; on Linux, add `-w 0` to prevent line-wrapping that breaks the header.
- **`status: "publish"` is irreversible without WP Admin access.** Default to `draft` and ask before publishing.
- **Page slug auto-generates from the title.** If the title has special characters, check that the generated slug is sensible after creation.

## Output

- Page edit URL and preview URL
- Pattern sequence used
- List of `IMAGE:` placeholders needing media (pass to `wp-add-media`)
- Sections requiring Lavon's personal input (e.g., specific dates, personal stories)

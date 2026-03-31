---
name: wp-add-media
description: Upload, optimize, and register images or video to the WordPress media library. Use when adding photos to the site, uploading a new profile picture, resolving IMAGE: placeholders left after wp-new-page, or when the user says "add this photo", "upload my picture", or "use this image on the site". Also handles bulk placeholder resolution after a page is created.
license: MIT
compatibility: Requires WordPress 6.4+, WP REST API with media upload permissions. cwebp recommended but not required.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-add-media

Upload and optimize images. Resolve `IMAGE:` placeholders across pages.

## Arguments

- `SOURCE` — a local file path, a public image URL, or `resolve-placeholders`
- `ALT_TEXT` — (optional) descriptive alt text. Generated from filename if omitted.

## Single image upload

### Step 1 — Validate the source

If a local file: check it exists and is an image (jpg, png, webp, gif, svg).
If a URL: fetch headers and confirm `Content-Type: image/*`.

### Step 2 — Optimize (skip for SVG and GIF)

Target: under 400KB for standard images, under 800KB for hero/full-width.
If the image is larger or not WebP:

```bash
# Convert to WebP with quality 85
cwebp -q 85 input.jpg -o output.webp
```

If `cwebp` is unavailable, note the original size and proceed without conversion.

### Step 3 — Upload via REST API

```bash
curl -s -X POST "$WP_SITE_URL/wp-json/wp/v2/media" \
  -H "Authorization: Basic $(echo -n "$WP_USER:$WP_APP_PASSWORD" | base64)" \
  -H "Content-Disposition: attachment; filename=\"<filename>\"" \
  -H "Content-Type: image/webp" \
  --data-binary @output.webp
```

Capture the response: extract `id`, `source_url`, `media_details.sizes.thumbnail.source_url`.

### Step 4 — Set alt text

```bash
curl -s -X POST "$WP_SITE_URL/wp-json/wp/v2/media/<id>" \
  -H "Authorization: Basic $(echo -n "$WP_USER:$WP_APP_PASSWORD" | base64)" \
  -H "Content-Type: application/json" \
  -d "{\"alt_text\": \"$ALT_TEXT\"}"
```

## Resolve IMAGE: placeholders

When `SOURCE` is `resolve-placeholders`:

### Step 1 — Scan all pages

```bash
curl -s "$WP_SITE_URL/wp-json/wp/v2/pages?per_page=100&status=publish,draft" \
  | jq '[.[] | {id, title: .title.rendered, content: .content.raw}]'
```

### Step 2 — Find all IMAGE: patterns

Extract every `IMAGE:<description>` string from all page content.
For each, search the media library for an existing match by alt text or filename.

### Step 3 — Report

List:
- **Resolved**: `IMAGE:<desc>` → media ID + URL (already uploaded, matched by description)
- **Unresolved**: `IMAGE:<desc>` — needs a file upload

For unresolved items, ask for the source files before proceeding.

### Step 4 — Replace in page content

For each resolved placeholder, update the page content:
Replace `IMAGE:<desc>` in the `wp:image` block with the actual `id` and `url`.
Update via REST API (full content body required).

## Gotchas

- **WordPress auto-generates thumbnails on upload.** If the media endpoint returns a 200 but the image doesn't appear in the library, check that the uploads directory is writable: `wp eval "echo wp_upload_dir()['error'];"`.
- **Application Passwords require HTTPS.** HTTP sites will silently fail auth on media uploads. Use the site's HTTPS URL.
- **Large files time out on shared hosting.** If an upload fails with a timeout, reduce image size or use WP-CLI: `wp media import <file> --post_id=<page_id> --featured_image`.
- **`Content-Type` must match the actual file format.** Uploading a JPEG with `Content-Type: image/webp` will create a corrupt entry.

## Output

For single upload: media ID, full URL, file size, dimensions.
For resolve-placeholders: table of resolved vs. unresolved with media IDs for resolved items.

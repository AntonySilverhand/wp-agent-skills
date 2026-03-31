---
name: wp-create-pattern
description: Create or update a named WordPress block pattern — a reusable, animated, responsive page section that becomes part of the site's design vocabulary. Use when building a new section type (hero, gallery, timeline, contact, etc.), updating an existing pattern, or when the user describes a visual section they want on their site. Produces production-grade markup using the site's design tokens and GSAP data-anim attributes.
license: MIT
compatibility: Requires WordPress 6.4+ with the wp-setup-theme FSE theme active. WP REST API must be accessible.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-create-pattern

Create or update a named block pattern. Patterns are the site's design vocabulary —
reusable sections assembled into pages.

## Pattern vocabulary

| Slug | Purpose |
|---|---|
| `hero-cinematic` | Full-viewport hero with parallax text |
| `about-split` | Two-column text + image (reversible) |
| `timeline-scroll` | Vertical milestone timeline |
| `gallery-masonry` | Animated masonry photo grid |
| `quote-feature` | Large pull-quote with decorative accent |
| `contact-minimal` | Clean contact form + social links |
| `nav-sticky` | Transparent → frosted glass nav on scroll |

For detailed visual specs on each pattern, read [references/pattern-guide.md](references/pattern-guide.md).

## Subagent strategy

When creating multiple patterns in one session, use subagents in parallel:
- Each pattern is fully independent — launch one subagent per pattern
- Each subagent reads theme.json, generates markup, and registers the pattern
- Particularly useful for the initial site build (all 7 vocabulary patterns at once)

For a single pattern, no subagents needed.

## Step 1 — Establish intent

If the slug is in the vocabulary above: use its defined purpose.
If it's new: ask what the section is for and what mood (dramatic / gentle / informative / playful).

## Step 2 — Read current theme.json

Fetch `theme.json` from the theme. **Only use CSS token variables** — never hardcode colors or fonts:
```
var(--wp--preset--color--accent)
var(--wp--preset--font-family--display)
var(--wp--preset--spacing--8)
```

## Step 3 — Design the pattern

Every pattern must:
- Be fully responsive (mobile-first)
- Use WordPress core blocks (Group, Columns, Cover, Image, Heading, Paragraph, Buttons)
- Include `data-anim` on all key visual elements (see CLAUDE.md for full list)
- Use spacing tokens — never hardcode pixel padding
- Include one visually memorable detail: a decorative element, unexpected layout, or typographic flourish. It should feel designed, not generic.

## Step 4 — Write the pattern file

```php
<?php
/**
 * Pattern: <Pattern Name>
 * Description: <description>
 * Categories: lavon-patterns
 * Slug: lavon/<pattern-slug>
 */
?>
<!-- wp:group {"className":"pattern-<slug>","layout":{"type":"constrained"}} -->
<div class="wp-block-group pattern-<slug>">
  <!-- blocks here -->
</div>
<!-- /wp:group -->
```

Register in `functions.php` (add to the existing block pattern section):
```php
register_block_pattern('lavon/<slug>', [
  'title'      => '<Pattern Name>',
  'categories' => ['lavon-patterns'],
  'content'    => file_get_contents(__DIR__ . '/patterns/<slug>.php'),
]);
```

## Step 5 — Verify

```bash
# Pattern appears in REST API
curl "$WP_SITE_URL/wp-json/wp/v2/block-patterns/patterns" | grep '"lavon/<slug>"'
```

## Gotchas

- **`register_block_pattern` must run after `init`**, not at the top level of `functions.php`. Wrap it in `add_action('init', function() { ... });` if it's not already.
- **Pattern slugs must be namespaced** (`lavon/<slug>`), not bare strings. Bare slugs will be accepted silently but break in the editor.
- **`file_get_contents` on a PHP file returns the raw PHP/HTML, not executed output.** This is correct — the block editor parses the HTML, not PHP. Don't use `include` or `require`.
- **`data-anim` attributes must be on the wrapper `<div>`, not the inner block `<!-- wp: -->` comment.** The comment is stripped; only the rendered HTML gets the attribute.
- **Block validator rejects unknown attributes on standard blocks.** Add custom attributes to a wrapper `wp:group` or `wp:html` block, not directly on a `wp:heading`.

## Output

Pattern slug, `data-anim` attributes used, first 20 lines of markup.
Suggest the next pattern to create.

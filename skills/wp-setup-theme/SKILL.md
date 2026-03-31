---
name: wp-setup-theme
description: Scaffold a WordPress Full Site Editing (FSE) theme for a personal website — including GSAP scroll animations, fluid design tokens, and a block pattern registry. Use when setting up a new WordPress site from scratch, rebuilding the theme, or when the user asks to initialise or scaffold the site theme. Activates via WP-CLI and WP REST API.
license: MIT
compatibility: Requires WP-CLI available on the server, WordPress 6.4+, and environment variables WP_SITE_URL, WP_USER, WP_APP_PASSWORD
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-setup-theme

Scaffold the full FSE theme with GSAP animation infrastructure, fluid design tokens, and block pattern registry. Run once at setup; safe to re-run (updates, doesn't overwrite content).

## Environment

```bash
export WP_SITE_URL="https://yoursite.com"
export WP_USER="admin"
export WP_APP_PASSWORD="xxxx xxxx xxxx xxxx"  # WP Admin → Users → Profile → Application Passwords
```

## Step 1 — Gather preferences

Ask for:
- Site owner's name
- Aesthetic: `ethereal` / `bold-editorial` / `minimalist` / `warm-organic`
- Body font (default: **DM Sans**), Display font (default: **Cormorant Garamond**)
- Tagline (one line)

## Step 2 — Generate theme.json

Use the token reference at [references/design-tokens.md](references/design-tokens.md). Pick the palette for the chosen aesthetic and generate `theme.json` with:
- 5-color palette (background, foreground, accent, accent-muted, surface)
- 8 fluid font sizes using `clamp()` (xs → 4xl)
- 10-step spacing scale (8-point grid)
- Content width 1200px, wide width 1440px

## Step 3 — Write animations.js

GSAP initialiser. Enqueued after GSAP + ScrollTrigger CDN scripts. Must:
- Register ScrollTrigger
- On DOMContentLoaded, query all `[data-anim]` and map to animations:

| `data-anim` | Effect |
|---|---|
| `fade-up` | y:40,opacity:0 → y:0,opacity:1 |
| `reveal-left` | x:-60,opacity:0 → x:0,opacity:1 |
| `reveal-right` | x:60,opacity:0 → x:0,opacity:1 |
| `parallax` | depth via `data-anim-depth` attr (default 0.3) |
| `text-split` | character stagger, y:-10,opacity:0 → y:0,opacity:1 |
| `counter` | innerText 0 → target number |

- All: `scrub: false`, `once: true`, trigger at `top 85%`
- Stagger delay via `data-anim-delay` attribute (seconds)
- **Honour `prefers-reduced-motion`**: skip all animations if set

## Step 4 — Write style.css

- Map `theme.json` tokens to CSS custom properties
- Reset: `box-sizing: border-box`, `margin: 0`, `scroll-behavior: smooth`
- Body: font and color from tokens
- Utilities: `.container`, `.container--wide`, `.sr-only`

## Step 5 — Write functions.php

Register the theme, enqueue GSAP from CDN, register `lavon-patterns` category:

```php
function lavon_scripts() {
    wp_enqueue_script('gsap', 'https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js', [], null, true);
    wp_enqueue_script('gsap-st', 'https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js', ['gsap'], null, true);
    wp_enqueue_script('lavon-anim', get_template_directory_uri() . '/animations.js', ['gsap-st'], null, true);
    wp_enqueue_style('lavon-style', get_stylesheet_uri());
}
add_action('wp_enqueue_scripts', 'lavon_scripts');

register_block_pattern_category('lavon-patterns', ['label' => 'Lavon']);
```

## Step 6 — Activate

```bash
wp theme activate <theme-slug>
wp option update blogname "<name>"
wp option update blogdescription "<tagline>"
```

## Step 7 — Verify

- Theme active in WP Admin → Appearance
- No GSAP errors in browser console
- Color tokens appear in block editor palette

## Gotchas

- **WP-CLI must run as the web server user** (or `--allow-root`), otherwise file permissions break.
- **CDN-loaded GSAP conflicts with local installs.** If a page builder or plugin already loads GSAP, dequeue theirs or deregister the CDN enqueue.
- **FSE themes require `"blockTemplates": true`** in `theme.json` settings, or the Site Editor won't load.
- **`prefers-reduced-motion` must be checked at runtime**, not at load time — the media query can change mid-session.
- **ScrollTrigger must be registered after gsap loads.** The `['gsap-st']` dependency array handles this; don't inline both scripts.

## Output

Report: theme slug activated, color palette (names + hex), fonts chosen, tagline set.
Suggest: "Run `wp-create-pattern` with `hero-cinematic` next."

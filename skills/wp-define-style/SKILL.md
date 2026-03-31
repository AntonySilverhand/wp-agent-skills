---
name: wp-define-style
description: Update the site's global design tokens — colors, typography, spacing — in theme.json. Use when the user wants to change the site's color scheme, switch fonts, adjust sizing, or restyle the entire site. A single run propagates the change everywhere. Examples of triggers: "make it darker", "change the accent to rose", "use a bigger body font", "switch to a serif headline font".
license: MIT
compatibility: Requires WordPress 6.4+ FSE theme (wp-setup-theme), write access to theme directory
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-define-style

Update global design tokens. All visual decisions in the site flow from `theme.json` — changing a token changes the entire site.

## Step 1 — Read current theme.json

Fetch the file. Display the current state:
- Color palette (slug + hex for each entry)
- Font families (body + display)
- Font sizes (slug + clamp value for xl, 2xl, 3xl)

## Step 2 — Interpret the change request

Map plain-English requests to specific token changes:

| User says | Change |
|---|---|
| "darker", "moodier" | background → deep dark, foreground → near-white, surface → slightly lighter dark |
| "warmer" | shift background toward warm off-white, accent toward amber/terracotta |
| "accent to [color]" | update `accent` hex; derive `accent-muted` at 60% opacity on background |
| "bigger type" | increase clamp values by ~20% across the scale |
| "serif headlines" | swap display font to Cormorant Garamond, Playfair Display, or EB Garamond |
| "more minimal" | reduce accent contrast, increase spacing scale, lighten surface |

## Step 3 — Check contrast before applying

For every color change, verify WCAG AA:
- `foreground` on `background`: minimum 4.5:1
- `accent` used as text: minimum 4.5:1 on background
- Large text (display font, xl+): minimum 3:1

Report the ratio. Flag any failures — don't apply a palette that fails contrast.

## Step 4 — Show diff, wait for confirmation

```
BEFORE → AFTER
background:  #F9F8F5 → #0F0F0F
foreground:  #2C2C2C → #F5F5F0
accent:      #B8A99A → #E8FF00
...
```

Ask: "Apply this change?" This is irreversible without git — confirm first.

## Step 5 — Write and deploy

Update `theme.json`. If a font changed, also update the `@import` in `style.css`.
For Google Fonts: use `@import url('https://fonts.googleapis.com/css2?family=...')`.

```bash
wp cache flush  # clear block editor cache so new tokens appear immediately
```

## Step 6 — Verify

Fetch the frontend and check the CSS custom properties match new values:
```bash
curl -s "$WP_SITE_URL" | grep "var(--wp--preset--color--accent)"
```

## Gotchas

- **Block-level color overrides survive a theme.json change.** If a pattern or page has inline `style="color:#..."`, it won't update. After a palette change, run `wp-update-section` to remove hardcoded overrides.
- **Google Fonts must be imported before `body` styles**, otherwise fallbacks flash on load.
- **`clamp()` values in fontSizes are ignored by the classic editor.** If Lavon ever uses the classic editor, add a matching `px` fallback.
- **The block editor caches `theme.json` aggressively.** `wp cache flush` alone is sometimes not enough — also clear browser cache or use Ctrl+Shift+R after updating.

## Output

Before/after token table. Contrast ratio for foreground/background and accent/background.
Note any patterns with hardcoded values that now conflict.

# wp-agent-skills

AI agent skills for managing a WordPress personal site — from zero to live with a single command.

Built for **Full Site Editing (FSE)** + **GSAP animations**, cross-OS (Windows/macOS/Linux).

---

## Install

```bash
npx skills add https://github.com/AntonySilverhand/wp-agent-skills
```

Or in Claude Code:
```
/plugin marketplace add AntonySilverhand/wp-agent-skills
```

Works with Claude Code, Cursor, Windsurf, Codex, Gemini CLI, OpenCode, and 40+ agents.

---

## Skills

| Skill | Purpose |
|-------|---------|
| `wp-install` | Detect OS, install WordPress (local SQLite or remote), configure credentials |
| `wp-setup-theme` | Scaffold a custom FSE theme with GSAP `animations.js` and `theme.json` tokens |
| `wp-create-pattern` | Create or update a named block pattern |
| `wp-define-style` | Set global colors, fonts, spacing in `theme.json` |
| `wp-new-page` | Assemble a new page from existing block patterns |
| `wp-update-section` | Surgically edit a pattern across all pages |
| `wp-add-media` | Upload, WebP-optimize, and register media |
| `wp-seo-pass` | Full SEO pass — meta, Open Graph, schema, alt text |
| `wp-health-check` | Parallel check: updates, performance, links, security, a11y |
| `wp-security` | Wordfence setup, author enumeration block, REST API hardening, file protection |

---

## Design workflow

These skills compose with the [Anthropic frontend-design skill](https://github.com/anthropics/skills) and [GreenSock GSAP skills](https://github.com/greensock/gsap-skills). Install them alongside:

```bash
npx skills add https://github.com/anthropics/skills
npx skills add https://github.com/greensock/gsap-skills
```

Then follow this sequence:

```
/theme-factory      → pick a palette + font pair
/frontend-design    → commit to a bold aesthetic direction
/wp-define-style    → encode into theme.json
/wp-setup-theme     → scaffold FSE theme with GSAP
/wp-new-page        → assemble pages from patterns
/wp-security        → harden before going live
/wp-health-check    → final QA pass
```

---

## Animation system

The theme's `animations.js` reads `data-anim` attributes — no GSAP code needed in patterns:

| Attribute | Effect |
|-----------|--------|
| `data-anim="fade-up"` | Fade in from below on scroll |
| `data-anim="reveal-left"` | Slide in from left |
| `data-anim="parallax"` | Parallax scroll depth |
| `data-anim="text-split"` | Character-by-character reveal |
| `data-anim="counter"` | Animated number counter |

---

## Requirements

- PHP 8.0+
- WP-CLI (auto-downloaded by `wp-install`)
- Local: PHP built-in server + SQLite (no MySQL needed)
- Remote: SSH or cPanel access

---

## License

MIT

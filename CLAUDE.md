# wp-agent-skills — Claude Code

These skills are installed globally and invoked by name. After installing via
`npx skills add`, each skill is available as a slash command in Claude Code.

## Available skills

- `wp-install` — Set up WordPress (local SQLite or remote)
- `wp-setup-theme` — Scaffold FSE theme with GSAP animations
- `wp-create-pattern` — Create/update block patterns
- `wp-define-style` — Update theme.json design tokens
- `wp-new-page` — Assemble pages from patterns
- `wp-update-section` — Edit a pattern across all pages
- `wp-add-media` — Upload and optimize media
- `wp-seo-pass` — Full SEO pass
- `wp-health-check` — Parallel site health check
- `wp-security` — Wordfence + hardening

## Pairing with design skills

These skills compose with `frontend-design`, `theme-factory`, and `gsap-*` skills.
Install them separately:
```bash
npx skills add https://github.com/anthropics/skills
npx skills add https://github.com/greensock/gsap-skills
```

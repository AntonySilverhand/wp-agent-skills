# wp-agent-skills — Agent Instructions

This repository contains AI agent skills for managing WordPress personal sites.
Skills follow the [agentskills.io](https://agentskills.io) SKILL.md format and
are compatible with Claude Code, Cursor, Windsurf, OpenAI Codex, Gemini CLI,
OpenCode, and 40+ agents.

## Skills overview

Each skill lives at `skills/<name>/SKILL.md` and covers one phase of the
WordPress site lifecycle:

| Skill | Phase |
|-------|-------|
| `wp-install` | Setup |
| `wp-setup-theme` | Theme |
| `wp-create-pattern` | Design |
| `wp-define-style` | Design |
| `wp-new-page` | Content |
| `wp-update-section` | Content |
| `wp-add-media` | Media |
| `wp-seo-pass` | SEO |
| `wp-health-check` | QA |
| `wp-security` | Security |

## Installing

```bash
npx skills add https://github.com/AntonySilverhand/wp-agent-skills
```

## Usage

After installing, invoke skills by name in your agent. Example:
- Claude Code: `/wp-install`
- Cursor: open command palette → skill name
- Codex: mention skill name in prompt

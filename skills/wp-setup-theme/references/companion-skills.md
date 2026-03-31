# Companion Skills for Design & Animation

These external skills pair directly with the wp-* skills for visual design,
typography, color, and GSAP animation work. Install them alongside this package.

## Install

```bash
npx skills add https://github.com/anthropics/skills
npx skills add https://github.com/greensock/gsap-skills
```

---

## Design skills (anthropics/skills)

| Skill | When to use |
|-------|-------------|
| `theme-factory` | Pick from 10 pre-built color + font pairs before running `wp-define-style` |
| `frontend-design` | Commit to a bold, distinctive aesthetic — avoids generic AI look. Use before building any new pattern or page |
| `brand-guidelines` | If Lavon has a brand identity to apply consistently |
| `canvas-design` | Build hero graphics, diagrams, or visual canvases |
| `algorithmic-art` | Generate background textures, geometric patterns, generative art for section backgrounds |

### Recommended design workflow

```
/theme-factory        → pick palette + font pair
/frontend-design      → commit to aesthetic direction
/wp-define-style      → encode tokens into theme.json
/wp-setup-theme       → scaffold FSE theme with GSAP
/wp-create-pattern    → build block patterns
```

---

## Animation skills (greensock/gsap-skills)

The theme's `animations.js` uses GSAP under the hood. These skills teach correct
GSAP usage when writing or debugging animation code.

| Skill | When to use |
|-------|-------------|
| `gsap-core` | Writing tweens: `gsap.to()`, `from()`, `fromTo()`, easing, stagger |
| `gsap-scrolltrigger` | Scroll-triggered animations — the engine behind all `data-anim` attributes |
| `gsap-timeline` | Sequencing complex multi-step animations |
| `gsap-plugins` | SplitText (`text-split`), Flip, ScrambleText, MorphSVG |
| `gsap-performance` | Keeping animations at 60fps — transforms over layout props, `will-change` |

### data-anim → GSAP mapping

| `data-anim` value | GSAP mechanism | Relevant skill |
|-------------------|---------------|----------------|
| `fade-up` | `gsap.from()` + ScrollTrigger | gsap-core + gsap-scrolltrigger |
| `reveal-left` | `gsap.from()` + ScrollTrigger | gsap-core + gsap-scrolltrigger |
| `parallax` | ScrollTrigger `scrub` | gsap-scrolltrigger |
| `text-split` | SplitText + stagger | gsap-plugins + gsap-core |
| `counter` | `gsap.to()` on a number target | gsap-core |

### GSAP licensing note

GSAP core + ScrollTrigger are free for all uses including commercial.
Club plugins (SplitText, ScrambleText, MorphSVG, etc.) require a paid license
for production. Free on localhost. See https://gsap.com/licensing/

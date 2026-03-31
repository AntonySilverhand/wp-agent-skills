# WP Design — Design Workflow for WordPress Sites

When working on **visual design, animations, or layout** for this WordPress site, use the following skills in sequence:

---

## Phase 1: Concept & Direction

**Start here when:** Planning a new page, section, or visual direction.

| Skill | When to Use |
|---|---|
| `/frontend-design` | Generate distinctive, production-ready design concepts for patterns or pages |
| `/design-consultation` | Get expert design feedback on direction, hierarchy, and visual balance |

---

## Phase 2: Visual Execution

**Start here when:** You have a direction and need to build it out.

| Skill | When to Use |
|---|---|
| `/ui-ux-pro-max` | Comprehensive UI/UX review and refinement |
| `/penpot-uiux-design` | Create detailed mockups in Penpot before implementing |
| `/canvas-design` | Build visual canvases, diagrams, or hero graphics |

---

## Phase 3: Animation & Motion

**Start here when:** Adding GSAP animations to patterns.

| Skill | When to Use |
|---|---|
| `/gsap-core` | Core GSAP animation setup and timeline creation |
| `/gsap-scrolltrigger` | Scroll-based animations (parallax, fade-ins, pinning) |
| `/gsap-timeline` | Complex sequenced animations |
| `/gsap-plugins` | Advanced effects (MorphSVG, SplitText, etc.) |
| `/gsap-performance` | Optimize animations for 60fps on mobile |
| `/ui-animation` | Fine-tune easing, duration, and micro-interactions |

---

## Phase 4: QA & Polish

**Start here when:** Design is implemented and needs review.

| Skill | When to Use |
|---|---|
| `/design-review` | Designer's eye QA on visual execution |
| `/plan-design-review` | Review design decisions made during planning |

---

## Not Applicable Here

- `/frontend-slides` — This skill is for slide decks and presentations, not WordPress sites

---

## Quick Start

**Building a new pattern?**
```
/frontend-design → /gsap-scrolltrigger → /design-review
```

**Refreshing an existing section?**
```
/design-consultation → /ui-ux-pro-max → /gsap-performance
```

**Creating a hero with complex motion?**
```
/canvas-design → /gsap-timeline → /ui-animation
```

---

## Integration with WP Skills

After design work is complete, use:
- `/wp-create-pattern` to register the pattern
- `/wp-define-style` to update global theme.json tokens
- `/wp-new-page` to assemble pages with the new pattern

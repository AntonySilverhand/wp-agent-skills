# Pattern Design Guide

Detailed animation and layout specs for each vocabulary pattern.

## hero-cinematic

- Full viewport height (`min-height: 100vh`)
- Background: large image (Cover block) or gradient
- Headline: `data-anim="text-split"` — character-by-character reveal
- Subtext: `data-anim="fade-up"` with `data-anim-delay="0.4"`
- CTA button: `data-anim="fade-up"` with `data-anim-delay="0.7"`
- Scroll indicator: small arrow bouncing at bottom (CSS `@keyframes bounce`)
- Text contrast: always ensure overlay darkens/lightens image sufficiently (scrim: `rgba(0,0,0,0.35)`)

## about-split

- Two equal columns on desktop, single column stacked on mobile
- Left column (text): `data-anim="reveal-left"`
- Right column (image): `data-anim="reveal-right"`
- Decorative accent: thin vertical line between columns (CSS `::before`, accent color, hidden on mobile)
- Image: use `border-radius: 2px` and subtle `box-shadow`
- Variation: `about-split--reversed` swaps image to left

## timeline-scroll

- Vertical timeline, centered connecting line
- Each item: date label (small caps, accent color) + title (display font) + body text
- Connecting line: SVG `<line>` with `stroke-dashoffset` animated by ScrollTrigger
- Each item: `data-anim="fade-up"` with index-based `data-anim-delay` (0.1s × index)
- Mobile: line shifts to left edge, items full width

## gallery-masonry

- CSS Grid with `grid-auto-flow: dense`, columns: `repeat(auto-fill, minmax(280px, 1fr))`
- Mix portrait (tall) and landscape (wide) images using `grid-row/column: span 2`
- Each image: `data-anim="fade-up"` with index-based delay
- Hover state: `transform: scale(1.03)`, overlay with image caption fades in
- Lightbox: optional — add `data-lightbox` attribute for click-to-expand
- Always include descriptive alt text

## quote-feature

- Large `"` character as decorative background (font-size: 20rem, opacity: 0.06, accent color)
- Quote text: large italic serif (display font, `font-style: italic`), `data-anim="text-split"`
- Attribution: small caps, body font, `data-anim="fade-up"`
- Background: surface color with subtle border or gradient edge

## contact-minimal

- Maximum whitespace — padding top/bottom: spacing-10
- Fields: Name, Email, Message — minimal styling, bottom-border only
- Each field: `data-anim="fade-up"` with staggered delay
- Submit button: outline style, fills on hover
- Social links row: icons + handle text, `data-anim="fade-up"`
- No heavy borders, no heavy shadows — let whitespace create the form

## nav-sticky

- Initial state: transparent background, white or light text
- On scroll past 80px: `backdrop-filter: blur(12px)`, background `rgba(bg-color, 0.85)`
- Transition: `background 0.3s ease, backdrop-filter 0.3s ease`
- Logo: left-aligned, SVG preferred
- Links: right-aligned, spaced with `gap: spacing-6`
- Mobile (< 768px): hamburger icon → full-screen overlay menu with staggered link reveal
- Active link: accent-color underline (2px, bottom offset)

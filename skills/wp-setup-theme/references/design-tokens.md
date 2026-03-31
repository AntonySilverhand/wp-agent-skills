# Design Token Reference

## theme.json structure

```json
{
  "version": 2,
  "settings": {
    "color": {
      "palette": [
        { "slug": "background", "color": "...", "name": "Background" },
        { "slug": "foreground", "color": "...", "name": "Foreground" },
        { "slug": "accent",     "color": "...", "name": "Accent" },
        { "slug": "accent-muted", "color": "...", "name": "Accent Muted" },
        { "slug": "surface",    "color": "...", "name": "Surface" }
      ]
    },
    "typography": {
      "fontFamilies": [
        { "slug": "body",    "fontFamily": "..., sans-serif", "name": "Body" },
        { "slug": "display", "fontFamily": "..., serif",      "name": "Display" }
      ],
      "fontSizes": [
        { "slug": "xs",  "size": "clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem)",  "name": "XS" },
        { "slug": "sm",  "size": "clamp(0.875rem, 0.8rem + 0.375vw, 1rem)",    "name": "SM" },
        { "slug": "md",  "size": "clamp(1rem, 0.9rem + 0.5vw, 1.125rem)",      "name": "MD" },
        { "slug": "lg",  "size": "clamp(1.125rem, 1rem + 0.625vw, 1.375rem)",  "name": "LG" },
        { "slug": "xl",  "size": "clamp(1.375rem, 1.2rem + 0.875vw, 1.875rem)","name": "XL" },
        { "slug": "2xl", "size": "clamp(1.875rem, 1.5rem + 1.875vw, 3rem)",    "name": "2XL" },
        { "slug": "3xl", "size": "clamp(2.5rem, 2rem + 2.5vw, 4.5rem)",        "name": "3XL" },
        { "slug": "4xl", "size": "clamp(3.5rem, 2.5rem + 5vw, 7rem)",          "name": "4XL" }
      ]
    },
    "spacing": {
      "spacingSizes": [
        { "slug": "1",  "size": "4px",   "name": "1" },
        { "slug": "2",  "size": "8px",   "name": "2" },
        { "slug": "3",  "size": "12px",  "name": "3" },
        { "slug": "4",  "size": "16px",  "name": "4" },
        { "slug": "5",  "size": "24px",  "name": "5" },
        { "slug": "6",  "size": "32px",  "name": "6" },
        { "slug": "7",  "size": "48px",  "name": "7" },
        { "slug": "8",  "size": "64px",  "name": "8" },
        { "slug": "9",  "size": "96px",  "name": "9" },
        { "slug": "10", "size": "128px", "name": "10" }
      ]
    },
    "layout": {
      "contentSize": "1200px",
      "wideSize": "1440px"
    }
  }
}
```

## Using tokens in CSS

```css
/* Colors */
var(--wp--preset--color--background)
var(--wp--preset--color--foreground)
var(--wp--preset--color--accent)
var(--wp--preset--color--accent-muted)
var(--wp--preset--color--surface)

/* Typography */
var(--wp--preset--font-family--body)
var(--wp--preset--font-family--display)
var(--wp--preset--font-size--xl)

/* Spacing */
var(--wp--preset--spacing--6)
```

## WCAG contrast requirements

- Normal text on background: minimum 4.5:1
- Large text (18pt+ or 14pt bold) on background: minimum 3:1
- UI components (borders, icons): minimum 3:1

Check contrast at: https://webaim.org/resources/contrastchecker/

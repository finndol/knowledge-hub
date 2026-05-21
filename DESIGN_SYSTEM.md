# Design System

Reference for tokens, conventions, and constraints. This document is the API for anyone (human or AI) building UI in this codebase. Read it before composing components.

## TL;DR rules

1. **No magic numbers.** Every dimension, color, and font-size comes from a token.
2. **No `px` values.** Everything in `rem` (or `em` when scoped to a font-size).
3. **Components consume semantic tokens only** — never primitives or raw hex. `bg-surface-page`, not `bg-neutral-50`. `text-primary`, not `text-neutral-900`.
4. **All text components must be leading-trim eligible.** Either use a semantic heading (`h1`–`h6`, `p`, `blockquote`) or apply a typography utility class (`text-h1`, `text-medium`, `u-text-style-h1`, etc.) — these are listed in the trim selector in `src/styles/global.css`. Adding a brand-new text utility? Add its class to the `:where(...)` selectors.
5. **Themes are complete.** Every semantic token is defined in `light`, `dark`, and `brand`. No fallbacks.

## File map

| File | Purpose |
|---|---|
| `src/styles/tokens.css` | Primitives, fluid scales, themed semantic tokens, `@theme` bindings, container utilities. |
| `src/styles/global.css` | Imports `tailwindcss` + tokens. Font-faces, leading trim, base element styles, focus ring, legacy `u-text-style-*` utilities. |
| `src/pages/design-system.astro` | Live demo of every token. |
| `DESIGN_SYSTEM.md` | This document. |

## Themes

Themes are scoped by `data-theme` on any ancestor (commonly `<html>`). Three values: `light` (default), `dark`, `brand`.

```html
<html data-theme="dark">…</html>
<!-- or scope locally -->
<section data-theme="brand">…</section>
```

Tailwind variants are wired:

```html
<div class="bg-surface-page theme-dark:bg-surface-raised">…</div>
```

## Leading-trim utility — read this before writing any text component

Half-leading above cap-height and below baseline is removed via `::before` and `::after` pseudo-elements on a curated selector list in `src/styles/global.css`. Offsets are font-specific:

- **Satoshi** (default `font-sans`): `0.34em` / `0.38em`.
- **Nib Pro** (opt in by adding `.font-accent`): overrides to `0.47em` / `0.29em`.

The trim eligibility selector includes:

- Semantic elements: `h1`–`h6`, `p`, `blockquote`.
- Legacy utilities: any class matching `[class*="u-text-style-"]`.
- New utilities: `.text-h1`–`.text-h6`, `.text-xlarge`, `.text-large`, `.text-medium`, `.text-small`.

**Directive for future text components:** apply one of the eligible classes, or render through a semantic element. If you invent a new text utility, add its class to the four `:where(...)` selectors in `global.css` — otherwise it will not trim and will look misaligned next to trimmed siblings.

Do not reimplement the trim. There is one canonical implementation.

## Fluid clamp formula

All fluid values interpolate linearly between two viewport anchors:

- **Min viewport:** `375px` (`23.4375rem`)
- **Max viewport:** `1440px` (`90rem`)
- **Range:** `66.5625rem`

Template (write per token — CSS custom properties don't compose reliably inside `clamp()` across browsers):

```css
clamp(
  MIN_REM,
  calc(MIN_REM + (MAX_REM - MIN_REM) * ((100vw - 23.4375rem) / 66.5625rem)),
  MAX_REM
)
```

To add a new fluid value: pick min and max in rem, paste the template, name the custom property, expose via `@theme` if it should be a Tailwind utility.

## Typography

| Token | Mobile (375px) | Desktop (1440px) | Tailwind utility | Semantic element |
|---|---|---|---|---|
| `--text-h1` | 2.5rem | 4.5rem | `text-h1` | `<h1>` |
| `--text-h2` | 2rem | 3.25rem | `text-h2` | `<h2>` |
| `--text-h3` | 1.625rem | 2.25rem | `text-h3` | `<h3>` |
| `--text-h4` | 1.375rem | 1.75rem | `text-h4` | `<h4>` |
| `--text-h5` | 1.125rem | 1.375rem | `text-h5` | `<h5>` |
| `--text-h6` | 1rem | 1.125rem | `text-h6` | `<h6>` |
| `--text-xlarge` | 1.125rem | 1.375rem | `text-xlarge` | — |
| `--text-large` | 1.0625rem | 1.1875rem | `text-large` | — |
| `--text-medium` | 1rem | 1.0625rem | `text-medium` | `<body>` default |
| `--text-small` | 0.875rem | 0.9375rem | `text-small` | — |

Larger headings taper more (bigger mobile/desktop delta); body sizes barely scale so reading rhythm stays stable.

**Line-height tokens** (tunable):

- `--leading-tight` (`1.1`) — all heading utilities.
- `--leading-body` (`1.5`) — all body utilities.

**Letter-spacing** is baked into semantic heading defaults and the `u-text-style-*` aliases (tighter for bigger sizes). It is not exposed as a token.

## Site margin and container

Page gutter scales `1rem` → `5rem` between 375px and 1440px viewports.

- `--site-margin` — the gutter value. Combined with a `90rem` max-width, content scales `20rem` → `90rem`.
- `u-container` — the standard page wrapper: `max-width: 90rem`, centered, with `padding-inline: var(--site-margin)`.
- `px-site` — one-off horizontal padding using the same value (no width cap, no centering).

```html
<main class="u-container">…</main>
```

## Spacing

### Micro spacing (padding, margin, gap)

Conservative fluid scaling. Available as Tailwind utilities: `p-xs`, `m-md`, `gap-lg`, `px-xl`, etc.

| Token | Mobile | Desktop |
|---|---|---|
| `--space-3xs` | 0.25rem | 0.25rem |
| `--space-2xs` | 0.5rem | 0.5rem |
| `--space-xs` | 0.75rem | 1rem |
| `--space-sm` | 1rem | 1.25rem |
| `--space-md` | 1.5rem | 2rem |
| `--space-lg` | 2rem | 3rem |
| `--space-xl` | 3rem | 4.5rem |
| `--space-2xl` | 4rem | 6rem |

### Tailwind v4 quirk: spacing tokens power all length utilities

In Tailwind v4 the spacing scale (`--spacing-*`) powers **every** length utility — not just padding/margin. That means `max-w-md`, `w-md`, `h-md`, `gap-md` all resolve to `var(--space-md)`. To prevent the misleading partial overlap with Tailwind's default `--container-*` named widths (where `max-w-md` would otherwise equal `28rem`), `tokens.css` clears the container namespace with `--container-*: initial;`.

Consequence: there is no built-in `max-w-md = 28rem`. For content-width caps you have three options:

1. `max-w-prose` (65ch — re-added explicitly in `tokens.css`), `max-w-narrow` (40rem), `max-w-wide` (75rem).
2. Arbitrary values: `max-w-[28rem]`.
3. Add a new reusable size via `--container-{name}` in the `@theme` block.

If you need a reusable named content size that doesn't already exist, add it to `tokens.css` rather than scattering `max-w-[…]` arbitrary values across files.

### Section padding (vertical rhythm)

Scales more aggressively. Use for top/bottom padding on page sections. Tailwind: `py-section-md`, `pt-section-lg`, etc.

| Token | Mobile | Desktop |
|---|---|---|
| `--section-sm` | 3rem | 5rem |
| `--section-md` | 4rem | 7rem |
| `--section-lg` | 5rem | 9rem |
| `--section-xl` | 6rem | 12rem |

## Colors — semantic tokens

Components only touch these. All three themes define every key.

### Surfaces

| Token | Tailwind | Use |
|---|---|---|
| `--surface-page` | `bg-surface-page` | Page background |
| `--surface-raised` | `bg-surface-raised` | Cards, panels |
| `--surface-sunken` | `bg-surface-sunken` | Wells, insets |
| `--surface-inverse` | `bg-surface-inverse` | Inverse sections within a theme |

### Text

| Token | Tailwind | Use |
|---|---|---|
| `--text-primary` | `text-primary` | Default body + heading color |
| `--text-secondary` | `text-secondary` | Supporting copy |
| `--text-muted` | `text-muted` | Meta, captions, hint text |
| `--text-inverse` | `text-inverse` | Text on inverse surfaces |
| `--text-brand` | `text-brand-text` | Brand-colored links / accents |

### Brand / accent

| Token | Tailwind | Use |
|---|---|---|
| `--brand-default` | `bg-brand` / `text-brand` | CTA primary, accent fills |
| `--brand-hover` | `bg-brand-hover` | Hover state of brand surfaces |
| `--brand-active` | `bg-brand-active` | Pressed state |
| `--brand-subtle` | `bg-brand-subtle` | Tinted brand-flavored backgrounds |

### Borders

| Token | Tailwind | Use |
|---|---|---|
| `--border-subtle` | `border-border-subtle` | Default dividers |
| `--border-strong` | `border-border-strong` | Emphasized borders |

### Interactive overlays

Component patterns layer these — they don't replace `:hover` colors.

| Token | Tailwind | Use |
|---|---|---|
| `--interactive-hover-overlay` | `bg-hover-overlay` | Semi-transparent hover scrim |
| `--interactive-active-overlay` | `bg-active-overlay` | Pressed scrim |

### Focus

| Token | Tailwind | Use |
|---|---|---|
| `--focus-ring` | `outline-focus-ring` | Focus ring color |
| `--focus-ring-offset` | — | Outline offset (matches surface) |

The base `:focus-visible` style in `global.css` already wires this up — most components don't need to touch focus directly.

## Colors — primitives (do not reference in components)

Defined only for internal mapping. Two ramps, 11 steps each.

- **Neutral** (`--neutral-50` … `--neutral-950`) — warm-leaning, editorial.
- **Brand** (`--brand-50` … `--brand-950`) — Antler red, anchored at `--brand-500: #E84E1B`.

## Accessibility notes

WCAG AA targets (verified by eye against `oklch()` luminance — re-check with a contrast tool before launch):

- **Light theme:** all `--text-*` on `--surface-page` and `--surface-raised` clear 4.5:1. `--text-muted` on `--surface-page` is the lowest contrast and should be reserved for non-critical meta text.
- **Dark theme:** all clear comfortably. `--text-muted` is `--neutral-400` (~6:1).
- **Brand theme:** `--text-muted` (`--brand-200`) on `--surface-page` (`--brand-900`) is the lowest at ~6:1. `--brand-default` text (used sparingly) is borderline — prefer it as a fill, not a text color, in this theme.

Focus rings: all three themes use a brand-family color that hits ≥3:1 against the page surface.

## Common recipes

```html
<!-- Page shell -->
<main class="u-container py-section-lg">
  <section class="py-section-md">…</section>
</main>

<!-- Card -->
<article class="bg-surface-raised text-primary p-md rounded-md border border-border-subtle">
  <h3>Card title</h3>
  <p class="text-secondary mt-sm">Body copy.</p>
</article>

<!-- Inline brand accent -->
<a href="#" class="text-brand-text hover:text-brand-hover">Read more</a>

<!-- Theme-scoped section inside a light page -->
<section data-theme="brand" class="bg-surface-page text-primary py-section-lg">
  <h2>Brand-themed CTA block</h2>
</section>
```

## Constraints — what NOT to do

- Don't use Tailwind's default color palette (`bg-red-500`, `text-gray-700`, etc.). Use semantic tokens.
- Don't write `style="font-size: 18px"` or `class="text-[18px]"`. Use a typography token.
- Don't add hover/active backgrounds by hardcoding a darker color. Use `bg-hover-overlay` or change to a token state (`bg-brand-hover`).
- Don't define new spacing values inline. If you need one that doesn't exist, add it to `tokens.css` first.
- Don't add `.dark` classes. Theme switching is `data-theme`.
- Don't write a new leading-trim implementation. Extend the eligibility selectors in `global.css` instead.

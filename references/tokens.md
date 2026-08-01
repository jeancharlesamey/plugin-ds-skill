# plugin-ds-skill — design tokens

Every component in this system is built exclusively from these CSS custom properties
(defined on `:root` in `assets/figma-plugin-ds.css`). When building a new component,
never hardcode a color, size, or font value that already has a token here — reuse the
token. That's the single rule that keeps a new component looking like it belongs.

## Color

| Token | Value | Use |
|---|---|---|
| `--blue` | `#18a0fb` | Primary accent, focus rings, links |
| `--purple` | `#7b61ff` | Accent |
| `--purple4` | `rgba(123,97,255,.4)` | Accent, muted |
| `--hot-pink` | `#ff00ff` | Accent |
| `--green` | `#1bc47d` | Success/accent |
| `--red` | `#f24822` | Destructive/error |
| `--yellow` | `#ffeb00` | Warning/accent |
| `--black` | `#000000` | Pure black |
| `--black8` | `rgba(0,0,0,.8)` | Primary text (positive application) |
| `--black8-opaque` | `#333333` | Opaque equivalent of black8 (for borders that can't use alpha) |
| `--black3` | `rgba(0,0,0,.3)` | Secondary/disabled text |
| `--black3-opaque` | `#B3B3B3` | Opaque equivalent of black3 |
| `--black1` | `rgba(0,0,0,.1)` | Hairline borders |
| `--white` | `#ffffff` | Surfaces, inverse text |
| `--white8` | `rgba(255,255,255,.8)` | Inverse text, secondary |
| `--white4` | `rgba(255,255,255,.4)` | Inverse text, muted |
| `--white2` | `rgba(255,255,255,.2)` | Dividers on dark surfaces |
| `--grey` | `#f0f0f0` | Background |
| `--silver` | `#e5e5e5` | Dividers, borders |
| `--hud` | `#222222` | Dark floating-menu background (select menu) |
| `--toolbar` | `#2c2c2c` | Dark toolbar background |
| `--hover-fill` | `rgba(0,0,0,.06)` | Generic hover background |
| `--blue3` | `rgba(24,145,251,.3)` | Text selection background |
| `--selection-a` / `--selection-b` | `#daebf7` / `#edf5fa` | Alternating selection rows |

## Typography

- Font stack: `--font-stack: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI',
  Roboto, Helvetica, Arial, sans-serif;` — Inter loads via a single `@import` at the
  top of `figma-plugin-ds.css`, with a native-UI-font fallback chain behind it. See
  `typography.md` for the Figma-plugin manifest setup this needs (or the fully
  offline self-hosted alternative).
- Sizes: `--font-size-xsmall: 11px` (default) · `--font-size-small: 12px` ·
  `--font-size-large: 13px` · `--font-size-xlarge: 14px`
- Weights: `--font-weight-normal: 400` · `--font-weight-medium: 500` ·
  `--font-weight-bold: 600`
- Line height: `--font-line-height: 16px` (xsmall/small) ·
  `--font-line-height-large: 24px` (large/xlarge)
- Letter spacing — every size has a **positive** (dark text on light bg) and
  **negative** (light text on dark bg) variant, because tighter/looser tracking
  reads differently against each background:
  `--font-letter-spacing-pos-xsmall: .005em` / `--font-letter-spacing-neg-xsmall: .01em`
  `--font-letter-spacing-pos-small: 0` / `--font-letter-spacing-neg-small: .005em`
  `--font-letter-spacing-pos-large: -.0025em` / `--font-letter-spacing-neg-large: .0025em`
  `--font-letter-spacing-pos-xlarge: -.001em` / `--font-letter-spacing-neg-xlarge: -.001em`

## Border radius

`--border-radius-small: 2px` (inputs, checkboxes) · `--border-radius-med: 5px` ·
`--border-radius-large: 6px` (buttons)

## Shadows

`--shadow-hud: 0 5px 17px rgba(0,0,0,.2), 0 2px 7px rgba(0,0,0,.15)` (floating dark
menus, e.g. the select menu) · `--shadow-floating-window: 0 2px 14px rgba(0,0,0,.15)`

## Spacing & sizing scale

One scale, reused for padding, margin, width, and height everywhere — never invent a
one-off pixel value if one of these fits:

| Token | Value |
|---|---|
| `--size-xxxsmall` | 4px |
| `--size-xxsmall` | 8px |
| `--size-xsmall` | 16px |
| `--size-small` | 24px |
| `--size-medium` | 32px *(the default row/control height — button, input, icon, etc.)* |
| `--size-large` | 40px |
| `--size-xlarge` | 48px |
| `--size-xxlarge` | 64px |
| `--size-xxxlarge` | 80px |

## Spacing utility classes

`assets/figma-plugin-ds.css` also ships Tailwind-style atomic utilities built on the
sizing scale above, useful for laying out a plugin UI without writing one-off CSS:

- Padding: `.p-{size}`, `.pt-{size}`, `.pr-{size}`, `.pb-{size}`, `.pl-{size}`
- Margin: `.m-{size}`, `.mt-{size}`, `.mr-{size}`, `.mb-{size}`, `.ml-{size}`
  (where `{size}` is `xxxsmall` / `xxsmall` / `xsmall` / `small` / `medium` / `large`
  / `xlarge` / `xxlarge` / `huge`, mapping to the scale above — note `huge` maps to
  `--size-xxxlarge`)
- Display: `.hidden` `.inline` `.block` `.inline-block` `.flex` `.inline-flex`
- Flex direction: `.column` `.column-reverse` `.row` `.row-reverse`
- Flex wrap/grow/shrink: `.flex-wrap` `.flex-wrap-reverse` `.flex-no-wrap`
  `.flex-shrink` `.flex-no-shrink` `.flex-grow` `.flex-no-grow`
- Justify/align: `.justify-content-{start|end|center|between|around}`,
  `.align-items-{start|end|center|stretch}`, `.align-content-{start|end|center|stretch}`,
  `.align-self-{start|end|center|stretch}`

## Known upstream quirk

A handful of components in the original library (`label`, `section-title`,
`input__field`, `textarea`, `disclosure__label`/`__content`,
`select-menu__label`/`__divider-label`) reference `var(--line-height)` — a token
that was never actually defined in `:root` (only `--font-line-height` and
`--font-line-height-large` exist). This is reproduced faithfully in
`assets/figma-plugin-ds.css` for fidelity with the original, and is harmless in
practice (the browser just ignores the invalid `line-height` declaration and
inherits normally). Don't copy this into new components — use
`var(--font-line-height)` or `var(--font-line-height-large)` instead, whichever
matches the font size being used.

## Icon color filters

Icons are drawn once in black (`#000`) and recolored in CSS with `filter`, since a
plain background-image can't take a `fill` override. When adding a *new* brand color
as an icon modifier, generate the matching filter recipe (e.g. via a
"CSS filter generator" that solves for a target hex from black) rather than guessing —
these are the recipes already in use:

```
icon--blue      invert(54%) sepia(16%) saturate(7499%) hue-rotate(179deg) brightness(98%) contrast(101%)
icon--purple    invert(40%) sepia(59%) saturate(4001%) hue-rotate(232deg) brightness(103%) contrast(102%)
icon--purple4   invert(72%) sepia(40%) saturate(660%) hue-rotate(202deg) brightness(103%) contrast(103%)
icon--hot-pink  invert(18%) sepia(90%) saturate(3347%) hue-rotate(293deg) brightness(116%) contrast(132%)
icon--green     invert(66%) sepia(39%) saturate(5382%) hue-rotate(114deg) brightness(102%) contrast(79%)
icon--red       invert(43%) sepia(56%) saturate(5632%) hue-rotate(349deg) brightness(97%) contrast(95%)
icon--yellow    invert(78%) sepia(86%) saturate(1608%) hue-rotate(1deg) brightness(107%) contrast(104%)
icon--black     invert(0%) sepia(0%) saturate(7500%) hue-rotate(117deg) brightness(109%) contrast(105%)
icon--black8    invert(0%) sepia(56%) saturate(25%) hue-rotate(137deg) brightness(105%) contrast(60%)
icon--black3    invert(100%) sepia(0%) saturate(698%) hue-rotate(219deg) brightness(66%) contrast(127%)
icon--white     invert(100%) sepia(100%) saturate(0%) hue-rotate(269deg) brightness(103%) contrast(104%)
icon--white8    invert(99%) sepia(2%) saturate(5%) hue-rotate(55deg) brightness(104%) contrast(98%)
icon--white4    invert(99%) sepia(2%) saturate(897%) hue-rotate(245deg) brightness(117%) contrast(93%)
```

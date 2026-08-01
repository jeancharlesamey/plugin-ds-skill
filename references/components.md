# plugin-ds-skill — component catalog

This is the exact, verbatim markup for every component in the figma-plugin-ds library
(originally by Thomas Lowry). Use this file as ground truth whenever a user asks for
one of these components — copy the markup pattern exactly, don't improvise a different
class structure. All classes here are already defined in `../assets/figma-plugin-ds.css`.

Table of contents: Button · Checkbox · Disclosure · Icon · Icon button · Input ·
Labels and sections · Onboarding tip · Radio button · Select menu · Switch ·
Textarea · Type

Two easy-to-misdiagnose gotchas worth knowing before you build anything real with
this:
- **Text/icons not showing up at all?** Check `typography.md` first — the font
  loads via a network `@import`, which sandboxed preview environments sometimes
  block, and a Figma plugin iframe blocks by default unless allowlisted. That's a
  documented tradeoff, not a bug in the CSS.
- **Rows visibly shrinking/warping once a scrollable container has enough content
  to overflow?** That's almost always a missing `flex-shrink: 0` on the
  container's children, not a bug in a specific component — see the comment above
  `.content` in `assets/starter.html` for the exact fix and why it happens.

---

## Button

```html
<!-- Primary -->
<button class="button button--primary">Label</button>
<button class="button button--primary-destructive">Label</button>
<button class="button button--primary" disabled>Label</button>

<!-- Secondary -->
<button class="button button--secondary">Label</button>
<button class="button button--secondary-destructive">Label</button>
<button class="button button--secondary" disabled>Label</button>

<!-- Tertiary (hyperlink-styled button) -->
<button class="button button--tertiary">Label</button>
<button class="button button--tertiary-destructive">Label</button>
<button class="button button--tertiary" disabled>Label</button>
```

| Modifier | Description |
|---|---|
| `button--primary` | Primary button |
| `button--primary-destructive` | Primary, red, for destructive actions (delete, etc.) |
| `button--secondary` | Secondary, outlined |
| `button--secondary-destructive` | Secondary, red outline |
| `button--tertiary` | Tertiary, styled like a hyperlink |
| `button--tertiary-destructive` | Tertiary, red |

No JS needed. `disabled` attribute handles the disabled visual state automatically.

---

## Checkbox

```html
<div class="checkbox">
  <input id="uniqueId" type="checkbox" class="checkbox__box">
  <label for="uniqueId" class="checkbox__label">Label</label>
</div>

<!-- checked -->
<div class="checkbox">
  <input id="uniqueId" type="checkbox" class="checkbox__box" checked>
  <label for="uniqueId" class="checkbox__label">Label</label>
</div>

<!-- disabled -->
<div class="checkbox">
  <input id="uniqueId" type="checkbox" class="checkbox__box" disabled>
  <label for="uniqueId" class="checkbox__label">Label</label>
</div>
```

Every checkbox needs a **unique `id`** matched to the label's `for` — the visible box is
drawn entirely with the label's `:before` pseudo-element (the real `<input>` is invisible
but still the actual clickable/focusable/keyboard-accessible control). No JS needed.

---

## Disclosure

Requires JS init. Rows collapse/expand and act like an accordion — opening one within
the same `<ul>` closes the others.

```html
<ul class="disclosure">
  <li class="disclosure__item disclosure--expanded">
    <div class="disclosure__label disclosure--section">Disclosure heading</div>
    <div class="disclosure__content">Panel content here</div>
  </li>

  <li class="disclosure__item disclosure--expanded"> <!-- expanded on load -->
    <div class="disclosure__label">Disclosure heading</div>
    <div class="disclosure__content">Panel content here</div>
  </li>

  <li class="disclosure__item">
    <div class="disclosure__label">Disclosure heading</div>
    <div class="disclosure__content">Panel content here</div>
  </li>
</ul>
```

```html
<script src="figma-plugin-ds.js"></script>
<script>
  disclosure.init();    // wire up all disclosure panels on the page
  // disclosure.destroy();  // tear down listeners
</script>
```

| Modifier | Applies to | Description |
|---|---|---|
| `disclosure--section` | `disclosure__label` | Styles the row like a section heading (bold) |
| `disclosure--expanded` | `disclosure__item` | Expanded by default |

---

## Icon

```html
<div class="icon icon--theme"></div>

<!-- recolored -->
<div class="icon icon--theme icon--blue"></div>

<!-- spinning (e.g. a loading spinner) -->
<div class="icon icon--spinner icon--spin"></div>

<!-- text used as an icon instead of an svg (e.g. "W" for width) -->
<div class="icon">W</div>
```

- `icon--{name}` selects which of the 86 bundled icons to render (full list in
  `icons.md`). All icons are already inlined as CSS data-URI backgrounds in
  `assets/figma-plugin-ds.css` — no separate image files are ever fetched.
- `icon--spin` adds an infinite rotation animation (pair with `icon--spinner`).
- `icon--{color}` recolors an icon via a `filter` recipe. Accepted colors: `blue`,
  `purple`, `purple4`, `hot-pink`, `green`, `red`, `yellow`, `black`, `black8`,
  `black3`, `white`, `white8`, `white4`.

---

## Icon button

A clickable wrapper around an `.icon`. The CSS alone gives it real `:hover`,
`:active`, and `:focus` styles — but a plain `<div>` can never actually receive
keyboard focus or a persisted "pressed" look on its own, so **call
`iconButton.init()`** any time an icon-button needs to be genuinely clickable
(not just decorative), the same way `disclosure`/`selectMenu` need their own
`.init()`.

```html
<!-- plain, one-shot action button (e.g. close, delete, share) — pair with
     your own click handler; iconButton.init() only adds keyboard support -->
<div class="icon-button">
  <div class="icon icon--close"></div>
</div>

<!-- toggle button (e.g. a toolbar tool-selection button) — the data-toggle
     attribute makes iconButton.init() also toggle icon-button--selected
     (+ aria-pressed) automatically on click AND on Enter/Space -->
<div class="icon-button" data-toggle>
  <div class="icon icon--blend"></div>
</div>

<!-- selected/active state, set up front (e.g. restoring saved state) -->
<div class="icon-button icon-button--selected" data-toggle aria-pressed="true">
  <div class="icon icon--blend"></div>
</div>

<!-- disabled — a plain <div> has no native :disabled, so use the modifier
     class. iconButton.init() also forces tabindex="-1" on it and skips
     wiring up any listeners, so it's fully inert, not just faded. -->
<div class="icon-button icon-button--disabled">
  <div class="icon icon--blend"></div>
</div>

<!-- disabled, using a real <button> instead of a <div> — :disabled works
     natively here, the modifier class isn't needed -->
<button class="icon-button" disabled>
  <div class="icon icon--blend"></div>
</button>
```

```html
<script src="figma-plugin-ds.js"></script>
<script>
  iconButton.init();    // adds tabindex/role to every .icon-button, wires up
                         // Enter/Space, and — only for [data-toggle] ones —
                         // toggles icon-button--selected + aria-pressed
  // iconButton.destroy();
</script>
```

| Attribute | Description |
|---|---|
| `data-toggle` | Opt in to automatic `icon-button--selected` toggling on click/keyboard. Omit it for a plain action button — otherwise it would visually stay "pressed" after a one-shot action, which is wrong. |
| `icon-button--selected` | The visual "on"/selected state. Set it upfront for initial state, or let `iconButton.init()` toggle it for you on `[data-toggle]` buttons. |
| `icon-button--disabled` | Disabled state for a `<div>`-based icon-button (use the native `disabled` attribute instead if it's a real `<button>`). Reuses the same `opacity: 0.3` fade every other disableable component in this kit uses — no new shade invented. Also gets `pointer-events: none` in CSS and a forced `tabindex="-1"` from `iconButton.init()`, so it's inert, not just dimmed. |

`iconButton.init()` only sets `tabindex`/`role` when they aren't already
present, so it's safe to call even if some icon-buttons already have their own
(e.g. a real `<button class="icon-button">` instead of a `<div>` — that works
too, and skips needing `tabindex`/`role` entirely since a `<button>` gets both
natively).

---

## Input

```html
<div class="input">
  <input type="input" class="input__field" placeholder="Placeholder">
</div>

<div class="input">
  <input type="input" class="input__field" value="Initial value">
</div>

<!-- disabled -->
<div class="input">
  <input type="input" class="input__field" value="Initial value" disabled>
</div>

<!-- with a leading icon -->
<div class="input input--with-icon">
  <div class="icon icon--angle"></div>
  <input type="input" class="input__field" value="Value">
</div>
```

| Modifier | Description |
|---|---|
| `input--with-icon` | Reserves space for a leading `.icon` inside the input |

---

## Labels and sections

```html
<div class="label">Label</div>

<div class="section-title">Section title</div>
```

`label` is muted/secondary text (e.g. a field caption). `section-title` is bold and
darker — used as a heading above a group of controls.

---

## Onboarding tip

```html
<div class="onboarding-tip">
  <div class="icon icon--styles"></div>
  <div class="onboarding-tip__msg">Onboarding tip goes here.</div>
</div>
```

---

## Radio button

```html
<div class="radio">
  <input id="radioButton1" type="radio" class="radio__button" value="Value" name="radioGroup">
  <label for="radioButton1" class="radio__label">Radio button</label>
</div>

<!-- checked -->
<div class="radio">
  <input id="radioButton2" type="radio" class="radio__button" value="Value" name="radioGroup" checked>
  <label for="radioButton2" class="radio__label">Radio button</label>
</div>

<!-- disabled -->
<div class="radio">
  <input id="radioButton3" type="radio" class="radio__button" value="Value" name="radioGroup" disabled>
  <label for="radioButton3" class="radio__label">Radio button</label>
</div>
```

Every radio in a group must share the same `name`. Every radio needs a unique `id`
matched to its label's `for`, same pattern as checkbox/switch.

---

## Select menu

Requires JS init. Progressively enhances a plain `<select>` into a custom Figma-style
dropdown — keep authoring it as a real `<select>` (accessible, form-submittable) and
let the JS build the visual menu around it.

```html
<!-- first option becomes the initial selection -->
<select id="uniqueId" class="select-menu">
  <option value="1">Item 1</option>
  <option value="2">Item 2</option>
  <option value="3">Item 3</option>
</select>

<!-- placeholder option (no value) leaves nothing selected initially -->
<select id="uniqueId" class="select-menu">
  <option>Please make a selection</option>
  <option value="2">Item 2</option>
  <option value="3">Item 3</option>
</select>

<!-- disabled -->
<select id="uniqueId" class="select-menu" disabled>
  <option value="1">Item 1</option>
  <option value="2">Item 2</option>
  <option value="3">Item 3</option>
</select>
```

```html
<script src="figma-plugin-ds.js"></script>
<script>
  selectMenu.init();    // build/rebuild the custom dropdown UI
  // selectMenu.destroy();
</script>
```

Behavior notes worth knowing before you touch this component:
- It auto-positions and auto-sizes so the open menu never overflows the plugin's
  iframe/viewport — it flips up or clamps height and scrolls as needed.
- `<optgroup>` elements become dividers automatically (with or without a `label`).
- Add an icon to the closed button by setting an `icon` attribute on the `<select>`
  itself, e.g. `<select class="select-menu" icon="icon--blend">`.
- It watches the underlying `<select>` with a `MutationObserver`, so if your app code
  adds/removes `<option>`s or toggles `disabled`/`value` programmatically, the visual
  menu re-syncs automatically — you don't need to call `.init()` again yourself.

---

## Switch

```html
<div class="switch">
  <input class="switch__toggle" type="checkbox" id="uniqueId">
  <label class="switch__label" for="uniqueId">Label</label>
</div>

<!-- checked -->
<div class="switch">
  <input class="switch__toggle" type="checkbox" id="uniqueId" checked>
  <label class="switch__label" for="uniqueId">Label</label>
</div>

<!-- disabled -->
<div class="switch">
  <input class="switch__toggle" type="checkbox" id="uniqueId" disabled>
  <label class="switch__label" for="uniqueId">Label</label>
</div>
```

Same hidden-input + label pattern as checkbox/radio: unique `id`, matching `for`.

---

## Textarea

```html
<textarea class="textarea" rows="2">Initial value</textarea>

<!-- disabled -->
<textarea class="textarea" rows="2" disabled>Initial value</textarea>
```

---

## Type

```html
<div class="type">UI11, size: xsmall (default), weight: normal, positive</div>
<div class="type type--large type--bold">UI13, size: large, weight: bold, positive</div>
<div class="type type--small type--medium type--inverse">UI12, size: small, weight: medium, negative</div>
```

| Modifier | Description |
|---|---|
| `type--small` | 12px |
| `type--large` | 13px |
| `type--xlarge` | 14px |
| `type--medium` | Medium weight |
| `type--bold` | Bold weight |
| `type--inverse` | Use when the text sits on a dark background (negative application) — adjusts letter-spacing |
| `type--inline` | `display: inline-block` instead of the block default |

Default (no modifiers): 11px, normal weight, positive (dark-on-light) application.

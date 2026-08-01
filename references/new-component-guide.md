# Building a new component in the figma-plugin-ds style

Use this process whenever a plugin or small website needs a UI piece that isn't
already in `components.md` — a tag/chip, a tab bar, a tooltip, a progress bar, a
segmented control, whatever the project calls for. The goal is that it looks and
behaves like it shipped with the original library, not bolted on.

## 1. Naming — BEM, kebab-case, one word for the block

Every existing component is a single kebab-case block name, with `__` for a child
element and `--` for a modifier. No camelCase, no nested blocks-within-blocks.

```
.tag { }              /* block */
.tag__remove { }      /* element — a child part of the block */
.tag--selected { }    /* modifier — a variant/state of the block */
```

Look at `.select-menu__item--selected` for how element and modifier combine — the
modifier always attaches to the piece it's modifying, not to the outer block.

## 2. Structure — only use tokens, never hardcode a value

Write the SCSS (or plain CSS, if the target project has no Sass build — see step 5)
by reaching for `references/tokens.md` for every single value: color, font-size,
weight, letter-spacing, padding, height, border-radius, shadow. If a value doesn't
exist in the token set, that's a signal to double check rather than invent a
one-off — but it's fine to add a new token to `:root` if genuinely nothing fits
(document why).

Standard control height is `var(--size-medium)` (32px) — match it unless there's a
specific reason not to (the select menu's internal row height of 30px is one of the
rare, deliberate exceptions).

Example — a "tag" component that doesn't exist in the library, built the same way
`.icon-button` and `.checkbox` were:

```css
.tag {
  display: inline-flex;
  align-items: center;
  height: var(--size-small);
  padding: 0 var(--size-xxsmall);
  border-radius: var(--border-radius-small);
  background-color: var(--grey);
  color: var(--black8);
  font-family: var(--font-stack);
  font-size: var(--font-size-xsmall);
  font-weight: var(--font-weight-normal);
  letter-spacing: var(--font-letter-spacing-pos-xsmall);
  line-height: var(--font-line-height);
  user-select: none;
}

.tag--selected {
  background-color: var(--blue);
  color: var(--white);
}

.tag__remove {
  margin-left: var(--size-xxxsmall);
  cursor: pointer;
}
```

## 3. Toggle-style inputs (checkbox/radio/switch pattern)

If the new component is a checkable control (a chip that toggles, a custom
checkbox variant, etc.), reuse the accessibility pattern from checkbox/radio/switch
instead of inventing click-handling JS:

- A real, visually-hidden native `<input>` (`opacity: 0`, small fixed hit box) —
  this keeps keyboard nav, screen readers, and form submission working for free.
- A sibling `<label for="...">` that draws the entire visible control using
  `:before` / `:after` pseudo-elements, styled via `:checked + .label`,
  `:disabled + .label`, etc. sibling selectors.
- Every instance needs a **unique `id`** matching the label's `for` — when
  generating markup programmatically, derive the id from something stable (a loop
  index, a data key) rather than a random string, so re-renders don't break the
  association.

This is a CSS-only pattern — no JS required, exactly like the existing
checkbox/radio/switch.

## 4. JS-driven components (disclosure/select-menu pattern)

If the component needs behavior beyond `:hover`/`:checked`/`:focus` (open/close
state, generating child DOM, positioning against the viewport), follow the same
shape as `disclosure` and `selectMenu`:

- One plain object with exactly two public methods: `init()` and `destroy()`.
  `init()` wires up (or, for `selectMenu`, actually builds) the DOM; `destroy()`
  removes the listeners/DOM it added. No other public surface.
- Plain `document.querySelectorAll` + `addEventListener` — no framework, no build
  step required to run it.
- If `init()` might be called more than once (e.g. after the page dynamically adds
  more instances), have it call `destroy()` first so listeners don't double up —
  see how `selectMenu.init()` does this.
- If the component's DOM can change after the fact from outside code (the caller
  adding/removing options, toggling `disabled`), use a `MutationObserver` inside
  `init()` to re-run the build automatically, the way `selectMenu` does — don't
  make the caller remember to re-initialize.
- Prefix all generated class names with the component's block name via a
  `const mySelector = 'my-component';` constant and string concatenation
  (`mySelector + '__item'`), rather than repeating the literal string everywhere.
- Ship it as an ES6 module (`export default myComponent;`) for bundler consumers
  *and* as an IIFE that assigns `window.myComponent = {...}` for script-tag/CDN
  consumers — that's why every component in `assets/figma-plugin-ds.js` is wrapped
  in `(function () { 'use strict'; ... })();`. If you're only shipping to one
  target (e.g. a single plugin's iframe with a `<script>` tag), the IIFE form
  alone is enough — see `assets/starter.html`.
- If the component has more than one *behavioral* mode (not just a visual
  modifier) — e.g. "this instance should persist a toggled state" vs. "this
  instance is a one-shot action" — gate the extra behavior behind a plain HTML
  attribute like `data-toggle`, checked inside `init()`, rather than adding a
  second public method or a config object. See `iconButton.init()` in
  `assets/figma-plugin-ds.js`: every `.icon-button` gets keyboard support
  unconditionally, but only ones marked `data-toggle` also get automatic
  `--selected`/`aria-pressed` toggling — a plain action button (close, delete)
  would look permanently "pressed" after one click if that weren't opt-in.

## 5. Cursor: pointer on the clickable element, default on its disabled state

Any element the user actually clicks — not just buttons, also a checkbox/radio/
switch's label, a disclosure row, a select-menu's trigger and its open rows —
should set `cursor: pointer`. It's cheap, easy to forget since the browser's
default arrow cursor doesn't look "broken," and it's one of the fastest visual
cues that something is interactive. Tie the reverse to whatever selector already
represents the disabled state: add `cursor: default` right there, next to the
`opacity: 0.3` / `color: var(--black3)` treatment that selector already applies —
don't leave a disabled control still showing a pointer cursor.

## 6. A clickable `<div>` needs help to actually be clickable

CSS `:hover`/`:active`/`:focus` rules are easy to write and easy to forget don't
do anything on their own for a non-native element. Before shipping any component
where a `<div>` or `<span>` (not a real `<button>`/`<input>`/`<a>`) is meant to be
interactive, make sure `init()` (or the static markup, if there's no JS layer)
covers all of these — `icon-button` is the reference example, and originally
shipped *without* this, which is exactly the bug class to avoid repeating:

- **`tabindex="0"`** — without it, the element is never in the tab order and can
  never receive `:focus` from anything other than `element.focus()` called by
  script. A mouse click alone does not focus a `<div>`.
- **`role="button"`** (or whatever ARIA role fits) — otherwise assistive tech has
  no idea the element is interactive.
- **A `keydown` handler for `Enter`/`Space`** that calls `element.click()` —
  native `<button>` elements get this for free; a `<div>` does not, so without it
  the component is mouse-only.
- **Real persisted state, not just `:active`** — `:active` only holds while the
  mouse button is physically down and releases the instant the user lets go, so
  it can never represent "this got selected." If the component needs a
  post-click state, that has to come from something durable: a CSS class
  toggled in a click handler (see `icon-button--selected` above), or — better,
  when the component is checkbox-like — the real hidden-`<input>`:`checked`
  pattern from section 3, which gets this for free.

## 7. Wiring it in

**Dropping into an existing HTML page (most plugin UIs and small sites):** append
the new component's CSS directly into `assets/figma-plugin-ds.css` (or a second
`<link>`ed stylesheet loaded after it — either works, since it's just CSS), and if
it needs JS, append the IIFE to `assets/figma-plugin-ds.js` or add a second
`<script>` tag. Call `.init()` next to the other `.init()` calls.

If the new component (or any existing one) lives inside a scrollable container —
a `flex: 1; overflow-y: auto;` panel like `.content` in `assets/starter.html` —
give that container's children `flex-shrink: 0`. Without it, once the content
overflows the available height, the browser's default `flex-shrink: 1` silently
compresses every row shorter than its intended height instead of just letting the
container scroll — components don't "break," they get visibly squashed and
misaligned (e.g. a switch's toggle knob, positioned with a fixed `top` value,
drifting off-center from its now-shorter label row). It only shows up once a real
page has enough rows to overflow, so it's easy to ship without ever noticing in a
short demo. See the comment above `.content` in `assets/starter.html` for the
exact rule.

**Extending the real, buildable npm package** (if the project actually is a fork or
build of figma-plugin-ds itself, not just a consumer of the compiled output): see
`source-mirror.md` for the SCSS/Gulp source layout — a new component gets its own
`src/styles/components/_{name}.scss` partial imported from the main
`figma-plugin-ds.scss` manifest, plus `src/js/modules/{name}.js` and
`src/js/iife/{name}.js` if it's interactive, then `npm run dev` (watch) or
`npm run build` (production) recompiles `dist/`.

## 8. Document it the same way

Add a section to `components.md` (or the project's own README, if the new component
lives in a fresh project) using the same shape as every other entry: a one-line
description, an HTML code block covering the default/checked/disabled states, a JS
init snippet if applicable, and a modifiers table. Consistency in the docs format
matters as much as consistency in the CSS — it's what makes the system feel like one
coherent thing instead of a pile of one-off widgets.

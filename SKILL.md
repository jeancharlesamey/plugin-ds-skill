---
name: plugin-ds-skill
description: Self-hosted mirror of the figma-plugin-ds UI kit (button, checkbox, switch, radio, input, textarea, label/section-title, onboarding-tip, icon/icon-button, disclosure, select-menu, type) — a lightweight CSS + vanilla-JS library that matches Figma's native UI. Use this whenever building a Figma plugin UI, or any small tool/website that should look and behave like Figma's own interface. Bundles the actual compiled CSS/JS/icons plus the design tokens and authoring process, so none of it requires opening, cloning, or reading the original figma-plugin-ds project folder. Trigger for requests like "build a Figma plugin UI", "add a button/dropdown/checkbox in the Figma style", "match figma-plugin-ds", or "create a new component that fits this design system" — for both reproducing an existing component exactly and designing brand-new ones in the same visual language.
---

# plugin-ds-skill — self-hosted figma-plugin-ds design system

**Rebooted & maintained by Jean-Charles Amey.**

This skill (`plugin-ds-skill`) is a complete, offline copy of the `figma-plugin-ds` library (originally
by Thomas Lowry): a small CSS + vanilla-JS kit that recreates Figma's own UI chrome,
meant for building Figma plugin interfaces (and, just as well, any small internal
tool or website that wants that same dense, native-feeling look). Everything needed
to use or extend it — compiled CSS, compiled JS, every design token, every
component's exact markup, and the process for authoring new components — lives in
this skill's own files. There's no dependency on the original project's repo or
folder; treat this skill as the source of truth.

## What's in here

```
assets/
  figma-plugin-ds.css   → drop-in compiled stylesheet (tokens + all components + all 86 icons inlined)
  figma-plugin-ds.js    → drop-in compiled script (disclosure/selectMenu/iconButton, each with .init()/.destroy())
  starter.html          → a MULTI-FILE plugin-UI skeleton — links the two files above via relative <link>/<script src>
  demo.html             → a SINGLE-FILE, fully self-contained demo — same UI, but with the CSS/JS inlined, not linked
references/
  components.md         → exact markup + modifiers for every existing component — READ THIS FIRST for any "add a ___" request
  tokens.md              → every color/spacing/type/radius/shadow token, plus the icon color-filter recipes
  typography.md           → how the Inter font is loaded (single @import), and the plain-website vs. Figma-plugin manifest setup
  icons.md                → the full, correct list of the 86 bundled icon names + how to draw a new one to match
  new-component-guide.md → the process for building a brand-new component that still looks/behaves native
  source-mirror.md       → the actual SCSS/Gulp/JS-module source, for forking the buildable npm package itself (rarely needed)
```

## Before anything else: is this a demo/preview or a real project?

This is the single most common way to break the styling when using this skill —
get this decision right first.

- **A demo, preview, artifact, "show me what this looks like", or anything that
  needs to render as ONE shareable file** → start from `assets/demo.html` and edit
  the body markup in place. It has the entire CSS and JS **inlined** in `<style>`
  and `<script>` tags. Never split a demo into separate `.css`/`.js` files linked
  by relative path — preview surfaces (chat previews, artifact viewers, sandboxed
  renderers) frequently can't resolve a second file sitting next to the HTML, so
  the page loads with no styling at all even though every path "looks" correct.
  One self-contained file has no path to resolve, so it renders identically
  everywhere: previewed, downloaded, or double-clicked later.
- **A real project directory that will live on disk** (an actual Figma plugin
  repo, a website's `src/`, anywhere with a real file tree and a person maintaining
  it) → use `assets/starter.html` plus the separate `figma-plugin-ds.css` /
  `figma-plugin-ds.js` files, linked normally. Separate files are the right call
  here — easier to diff, cache, and update independently — because relative paths
  reliably resolve when the files actually sit next to each other on a real
  filesystem.

If there's any doubt which situation applies, default to the single self-contained
file (`demo.html`'s approach) — it always works, whereas split files only work
under the right conditions.

## How to use this skill

**Implementing an existing component** (button, checkbox, switch, radio, input,
textarea, label, section-title, onboarding-tip, icon, icon-button, disclosure,
select-menu, type): open `references/components.md`, copy its markup pattern
exactly — same element structure, same class names, same modifier naming — and fill
in the project's actual content. Don't improvise a different class structure for
something that already has one defined here.

**Starting a brand-new plugin or small website that should use this system:**
1. Real project on disk → copy `assets/figma-plugin-ds.css` and
   `assets/figma-plugin-ds.js` into it, and copy `assets/starter.html` as the entry
   point (adjust its layout `<style>` block — the component markup inside it can
   stay as-is). Single-file demo/preview → copy `assets/demo.html` instead and edit
   its body markup directly; leave the inlined `<style>`/`<script>` blocks alone.
2. If using the multi-file form: link the CSS in `<head>`, the JS before `</body>`.
3. Call `disclosure.init()` and/or `selectMenu.init()` after the script tag if the
   page uses either of those two components. Also call `iconButton.init()` if the
   page has any `.icon-button` that's actually clickable (not purely decorative) —
   without it, the button can never receive keyboard focus and, for toggle-style
   buttons, never keeps a "selected" look after a click. Every other component is
   pure CSS, no init call needed.
4. Build the UI out of the patterns in `references/components.md`.
5. If this is going into a real Figma plugin (not just a website), read
   `references/typography.md` — the font is loaded via a one-line CSS `@import`,
   which needs two domains allowlisted in the plugin's `manifest.json` (or can be
   swapped for a fully offline self-hosted setup, also documented there).

**Creating a new component that isn't in the catalog** (a tag, tab bar, tooltip,
progress bar, whatever the project needs): read `references/new-component-guide.md`
for the naming convention, the token-first CSS approach, the toggle-input
accessibility pattern (checkbox/radio/switch), and the `init()`/`destroy()` JS
pattern for anything interactive — then check `references/tokens.md` while writing
the CSS so every value reuses an existing token instead of a one-off. This is what
keeps a new component indistinguishable from the ones that shipped with the
original library.

**Needing a new icon:** check `references/icons.md` for the full list first (86
already exist, all pre-inlined in the CSS). If a needed icon truly isn't there,
that file also has the exact drawing spec (32×32 canvas, single black fill,
evenodd) and how to encode + add it.

**Forking the actual buildable npm package** (Sass + Gulp pipeline, not just
consuming the compiled output): see `references/source-mirror.md`.

## Design philosophy to carry into new work

Everything in this system is deliberately plain: no framework, no build step
required to consume it, one flat design-token layer, BEM class names, and — for the
components that need behavior — the smallest possible vanilla-JS surface
(`init()` / `destroy()`, nothing else public, opt-in behavior gated behind a plain
HTML attribute like `data-toggle` rather than a second JS API). When extending it, resist the urge to
reach for a framework, a CSS-in-JS solution, or a component with a large
configurable API — a new piece should feel like it could have shipped in the
original library on day one.

# plugin-ds-skill 
Lightweight components for building plugins using Claude

## Contents
- [Intro](#intro)
- [Components Demo](#components-demo)
- [Installing the skill](#installing-the-skill)
- [What you can ask Claude](#what-you-can-ask-claude)
- [What's inside the skill](#whats-inside-the-skill)
- [MIT License](#mit-license)

## Intro

`plugin-ds-skill` is a skill that packages lightweight components for building plugins using Claude.
Originally started as a Figma plugin ds kit by Thomas Lowry; rebooted as skill & maintained by Jean-Charles Amey.

It's a complete, offline copy — compiled CSS, compiled JS, every design token, every component's exact markup, and the process for building new components in the same style — so Claude never needs to open or read this project folder to use it. Treat the skill as the source of truth; this folder stays untouched.

## Components Demo
Demo enerated with the skill.

See `assets/demo.html`
or
[online demo](https://claude.ai/public/artifacts/506236f4-5904-4f96-b6f7-ff12b2308d80)

## Installing the skill

Open `plugin-ds-skill.skill` and use "Save skill" to install it. Once installed, just ask Claude to build a plugin, add a plugin component in a lightweight style, it will trigger automatically. If not please, [create a new issue](https://github.com/jeancharlesamey/plugin-ds-skill/issues/new), and include your prompt.

## What you can ask Claude

- **Build a full Figma plugin UI** — buttons, inputs, dropdowns, checkboxes, switches, radio buttons, accordions, icon buttons, tooltips/onboarding tips — that looks and behaves exactly like Figma's own interface.
- **Add a single component** to an existing project ("add a dropdown in the Figma style") — Claude copies the exact markup/classes from the catalog rather than improvising a new structure.
- **Design a brand-new component** that isn't in the library (a tag, tab bar, progress bar, whatever the project needs) — Claude follows the same naming, token, and accessibility conventions so it looks like it shipped with the original kit.
- **Preview/demo a component or a whole screen** as one self-contained HTML file (works in chat previews, artifacts, or double-clicked locally) — or scaffold a real multi-file plugin project, depending on what you're doing.
- **Add a new icon** in the same visual style (32×32, single black fill) if one of the 86 bundled icons doesn't cover what's needed.
- **Fork the actual buildable npm package** (Sass + Gulp source, not just the compiled output) if the project needs to extend the library itself.

## What's inside the skill

```
assets/
  figma-plugin-ds.css   → drop-in compiled stylesheet — every token, every component, all 86 icons inlined
  figma-plugin-ds.js    → drop-in compiled script — disclosure / selectMenu / iconButton, each with .init()/.destroy()
  starter.html           → multi-file plugin-UI skeleton (links the two files above) — for a real project on disk
  demo.html               → single-file, fully self-contained demo exercising every component in every state — for previews/artifacts
references/
  components.md            → exact markup + modifiers for every component — the first stop for "add a ___"
  tokens.md                → every color / spacing / type / radius / shadow token, plus icon color-filter recipes
  typography.md            → how the Inter font loads (single CSS @import) and the Figma-plugin manifest requirement
  icons.md                 → the full, correct list of all 86 bundled icon names + how to draw a new one to match
  new-component-guide.md   → the process for building a brand-new component that still looks/behaves native
  source-mirror.md         → the real SCSS/Gulp/JS-module source, for forking the buildable npm package (rarely needed)
```

`SKILL.md` (inside the skill package) is the entry point Claude reads first —
it explains when to use `assets/demo.html` vs. `assets/starter.html`, when to call
`disclosure.init()` / `selectMenu.init()` / `iconButton.init()`, and points
to the right reference file for each kind of request.


## MIT License

See `LICENSE`
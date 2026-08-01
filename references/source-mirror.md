# plugin-ds-skill — buildable source mirror

`assets/figma-plugin-ds.css` / `assets/figma-plugin-ds.js` are the **compiled
output** — perfect for dropping into a plugin UI or website directly. This file is
the **source** those were built from: the actual SCSS partials, JS modules, and
Gulp pipeline. Reach for this only when the task is to *extend the real npm package*
(a fork of figma-plugin-ds itself, or a project that wants its own Sass build rather
than consuming static CSS) — for everything else, `components.md` +
`new-component-guide.md` + the compiled assets are all that's needed.

## Project layout

```
figma-plugin-ds/
├── package.json
├── gulpfile.js
├── index.js                          → re-exports the ES6 modules
├── src/
│   ├── icons/*.svg                   → 86 raw icon files
│   ├── styles/
│   │   ├── figma-plugin-ds.scss      → import manifest (order matters)
│   │   ├── base/
│   │   │   └── _figma-plugin-css-vars-utilities.scss
│   │   └── components/
│   │       └── _{button,checkbox,disclosure,icon,icon-button,input,
│   │              label,onboarding-tip,radio,section-title,select-menu,
│   │              switch,textarea,type}.scss
│   └── js/
│       ├── modules/{disclosure,selectMenu}.js   → ES6, `export default`
│       └── iife/{disclosure,selectMenu}.js      → same logic, `window.x = {...}`
└── dist/                              → gulp output (what assets/ mirrors)
    ├── figma-plugin-ds.css
    ├── iife/figma-plugin-ds.js
    └── modules/{disclosure,selectMenu}.js
```

## package.json

```json
{
  "name": "figma-plugin-ds",
  "devDependencies": {
    "autoprefixer": "^9.7.6",
    "gulp": "^4.0.2",
    "gulp-concat": "^2.6.1",
    "gulp-css-svg": "^1.3.1",
    "gulp-postcss": "^8.0.0",
    "gulp-prettier": "^3.0.0",
    "gulp-sass": "^4.1.0"
  },
  "dependencies": {},
  "scripts": {
    "dev": "gulp",
    "build": "gulp --production"
  },
  "description": "A UI library with CSS and vanilla JS that match the Figma UI for building plugins.",
  "main": "index.js",
  "style": "dist/figma-plugin-ds.css"
}
```

## gulpfile.js

```js
const { src, dest, watch, series, parallel } = require('gulp');
const sass = require('gulp-sass');
const concat = require('gulp-concat');
const postcss = require('gulp-postcss');
const autoprefixer = require('autoprefixer');
const cssSvg = require('gulp-css-svg');
const prettier = require('gulp-prettier');

const files = {
    scssPath: 'src/styles/**/*.scss',
    jsPathIIFE: 'src/js/iife/**/*.js',
    jsPathES6: 'src/js/modules/**/*.js',
    assetsPath: 'src/icons/**/*.svg'
}

// compiles SCSS -> CSS, autoprefixes, and inlines every icons/*.svg
// reference as a data-URI (this is what makes dist/*.css self-contained)
function scssTask(){
    return src(files.scssPath)
        .pipe(sass({ outputStyle: 'expanded' }))
        .pipe(postcss([autoprefixer()]))
        .pipe(cssSvg())
        .pipe(dest('dist'));
}

// concatenates every src/js/iife/*.js file into one dist/iife/figma-plugin-ds.js
function jsTaskIIFE(){
    return src([files.jsPathIIFE])
        .pipe(concat('figma-plugin-ds.js'))
        .pipe(prettier({
            parser: 'babel', printWidth: 100, tabWidth: 4, useTabs: true,
            semi: true, singleQuote: true, trailingComma: 'none',
            bracketSpacing: true, jsxBracketSameLine: true, arrowParens: 'always'
        }))
        .pipe(dest('dist/iife'));
}

// formats each ES6 module and copies it (unconcatenated) to dist/modules
function jsTaskES6(){
    return src([files.jsPathES6])
        .pipe(prettier({
            parser: 'babel', printWidth: 100, tabWidth: 4, useTabs: true,
            semi: true, singleQuote: true, trailingComma: 'none',
            bracketSpacing: true, jsxBracketSameLine: true, arrowParens: 'always'
        }))
        .pipe(dest('dist/modules'));
}

function watchTask(){
    watch([files.scssPath, files.jsPathIIFE, files.jsPathES6, files.assetsPath],
        { interval: 1000, usePolling: true },
        series(parallel(scssTask, jsTaskIIFE, jsTaskES6))
    );
}

// `npm run dev`   -> build once, then watch
// `npm run build` -> gulp --production (same tasks, no watch)
exports.default = series(
    parallel(scssTask, jsTaskIIFE, jsTaskES6),
    watchTask
);
```

## index.js

```js
import disclosure from './dist/modules/disclosure';
import selectMenu from './dist/modules/selectMenu';

export { disclosure, selectMenu };
```

## src/styles/figma-plugin-ds.scss (import manifest — order matters: base first)

```scss
@import 'base/_figma-plugin-css-vars-utilities';

@import 'components/button';
@import 'components/checkbox';
@import 'components/disclosure';
@import 'components/icon';
@import 'components/icon-button';
@import 'components/input';
@import 'components/label';
@import 'components/onboarding-tip';
@import 'components/radio';
@import 'components/section-title';
@import 'components/select-menu';
@import 'components/switch';
@import 'components/textarea';
@import 'components/type';
```

Adding a new component to the real build: create
`src/styles/components/_{name}.scss`, add `@import 'components/{name}';` to this
manifest (keep the list alphabetical), then `npm run dev`.

## src/styles/base/_figma-plugin-css-vars-utilities.scss

This is the token file — reproduced in full, human-readable form in
`tokens.md`. The SCSS source is identical to the compiled `:root` block at the top
of `assets/figma-plugin-ds.css`, plus the global reset (`* { box-sizing: border-box; }`,
`body { margin: 0; ... }`), the font `@import` (see `typography.md` for the
Figma-plugin manifest setup it needs, or the offline self-hosted alternative), and
the padding/margin/flex utility classes. Copy that block directly
if you need the raw SCSS — it compiles to plain CSS with no Sass-specific syntax
(no mixins, no functions), so it's byte-for-byte portable between the two files.

## Component SCSS partials

Every file under `src/styles/components/` uses plain SCSS nesting (`&__element`,
`&--modifier`) and nothing else Sass-specific — `.button` is the one exception,
using `@extend` twice (`.button--secondary-destructive` extends
`.button--secondary`; `.button--tertiary-destructive` extends `.button--tertiary`)
to avoid repeating the shared base styles for the destructive variants. Every rule
in every partial is already reproduced, flattened, in `assets/figma-plugin-ds.css`
— that compiled version is the fastest way to read "what does this component's CSS
actually do," since `&`-nesting can be harder to scan at a glance. The nested SCSS
form is only relevant if you're doing the same in a Sass project of your own.

## JS module pattern

Both `src/js/modules/disclosure.js` and `src/js/modules/selectMenu.js` (ES6,
`export default`) and their `src/js/iife/*.js` twins (same logic, wrapped in
`(function () { 'use strict'; ... window.x = {...} })();`) are reproduced in full
in `assets/figma-plugin-ds.js` (the IIFE form — that's the one that works with a
plain `<script>` tag, which covers the overwhelming majority of Figma plugin UIs
and small websites). See `new-component-guide.md` section 4 for the shape to follow
when writing a new one (`init()`/`destroy()`, MutationObserver for reactive
components, selector-prefix constants).

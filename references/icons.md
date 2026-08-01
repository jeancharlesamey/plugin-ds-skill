# plugin-ds-skill — icon list

86 icons ship with this system, each already inlined as a CSS data-URI background
image in `assets/figma-plugin-ds.css` under a `.icon--{name}` class — use them with
`<div class="icon icon--{name}"></div>` (see `components.md` for the full Icon
component pattern, including recoloring and the spin animation).

This list is generated from the actual CSS source (not the upstream project's
README, which has one stale entry: it lists `icon--updown`, but the real class — and
the one in this bundle — is `icon--up-down`). Use the names below.

```
adjust                          instance                        share
alert                           key                              smiley
angle                           layout-align-bottom             sort-alpha-asc
arrow-left-right                align-horizontal-centers        sort-alpha-dsc
up-down                         align-left                       sort-top-bottom
auto-layout-horizontal          align-right                      spacing
auto-layout-vertical            align-top                        spinner
back                            align-vertical-centers          star-off
blend-empty                     layout-grid-columns             star-on
blend                           layout-grid-rows                stroke-weight
break                           layout-grid-uniform             styles
caret-down                      library                          swap
caret-left                      link-broken                     theme
caret-right                     link-connected                  tidy-up-grid
caret-up                        list-detailed                   tidy-up-list-horizontal
check                           list-tile                       tidy-up-list-vertical
close                           list                             timer
component                       lock-off                        trash
corner-radius                   lock-on                         vertical-padding
corners                         minus                            visible
distribute-horizontal-spacing   play                             warning-large
distribute-vertical-spacing     plus                             warning
draft                           random
effects                         recent
ellipses                        resize-to-fit
eyedropper                      resolve-filled
forward                         resolve
frame                           reverse
group                           search-large
hidden                          search
horizontal-padding              settings
hyperlink
image
```

## Drawing a new icon in the same style

If a plugin or site needs an icon that isn't in this list, draw it to match the
existing set exactly rather than dropping in a differently-styled icon (e.g. a
two-tone icon pack or a stroked/outline style would visually clash):

1. **Canvas**: `viewBox="0 0 32 32"`, `width="32" height="32"`.
2. **Fill**: single flat color, `fill="#000"` (it gets recolored later via the
   `icon--{color}` filter modifiers — never hardcode a final color into the SVG).
3. **Style**: solid geometric fills, no strokes, `fill-rule="evenodd"` /
   `clip-rule="evenodd"` for shapes with holes (rings, letter counters). Look at
   `icon--check` or `icon--close` in the CSS for a minimal example, and
   `icon--settings` for a more complex one.
4. **Encoding**: convert the finished SVG into a CSS data-URI and add it as a new
   rule in the Icon section of `assets/figma-plugin-ds.css`:
   ```css
   .icon--my-new-icon {
     background-image: url("data:image/svg+xml;charset=utf8,<url-encoded-svg>");
   }
   ```
   (URL-encode `<`, `>`, `"`, `#` at minimum — swap double quotes for single quotes
   inside the SVG attributes first, that's what keeps the encoded string short and
   matches the house style seen throughout this file.)
5. Add the new name to the list above so future lookups find it.

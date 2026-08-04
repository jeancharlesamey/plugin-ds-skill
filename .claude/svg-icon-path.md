# Adding a new icon (when the user pastes a raw SVG)

Follow this checklist exactly, in order. No exceptions — do not silently
work around a mismatch, do not guess on the user's behalf.

## 1. Get a name
If the user didn't give the icon a name, **ask for one before doing anything
else**. Never invent, abbreviate, or guess a name from the SVG's shape.

## 2. Validate the SVG against house style
Every icon in this set must match `references/icons.md`'s spec:
- `viewBox="0 0 32 32"`, `width="32" height="32"`.
- Single flat fill (`fill="#000"` / `black`) — **no `stroke`, no `fill="none"`
  on shapes** (`fill="none"` is only allowed on the outer `<svg>` tag itself).
- `fill-rule="evenodd"` / `clip-rule="evenodd"` for any shape with a hole
  (rings, letter counters, etc.).

If the pasted SVG fails any of these checks (wrong canvas size, uses
strokes, multiple colors, missing viewBox, etc.):
- **Do not redraw, convert, or "fix" it yourself.**
- **Do not silently accept it as-is.**
- Tell the user exactly what's wrong and ask them to adjust and re-paste it.
  Repeat until it conforms — no exception, however small the deviation looks.

Only proceed to step 3 once the SVG passes every check above.

## 3. Wire the icon into all four places
Missing any one of these is an incomplete job — they were all missed once
before (the icon was added to the CSS/markup but not to the JS grid array,
so it silently never showed up in the demo's "All N icons" gallery, and the
person adding it had no way to find it).

1. **`assets/figma-plugin-ds.css`** — add `.icon--{name} { background-image:
   url("data:image/svg+xml;charset=utf8,<url-encoded-svg>"); }` in
   alphabetical position among the existing `.icon--*` rules. Encode `<`,
   `>`, `#` (as `%3C`, `%3E`, `%23`); use single quotes inside the SVG so no
   double-quote escaping is needed. Wrap multiple same-fill shapes in a
   single `<g fill='#000'>` to keep it compact.
2. **`assets/demo.html`** — mirror the *exact same* CSS rule into its
   inlined `<style>` block (same alphabetical spot).
3. **`assets/demo.html` — the `ICON_NAMES` JS array** (search for
   `var ICON_NAMES = [`, feeds the `#iconGrid` "All N icons" gallery). Add
   the new name in alphabetical position. **This is the actual visual test
   page — if it's not in this array, it will not render anywhere
   findable, even if the CSS rule is correct.** Also bump the "All N icons"
   section-title count.
4. **`references/icons.md`** — add the name to the plain-text icon list and
   bump the icon-count sentence at the top of the file.

## 4. Verify before reporting done
Two concrete, reusable checks — run both, don't skip either:

**a. Decoded SVG is well-formed XML** (catches encoding mistakes before the
user ever opens a browser):

```python
import re, urllib.parse, xml.dom.minidom as md

css = open('assets/figma-plugin-ds.css').read()
name = 'ICON_NAME_HERE'
m = re.search(
    r"\.icon--" + re.escape(name) + r" \{ background-image: url\(\"data:image/svg\+xml;charset=utf8,(.*?)\"\); \}",
    css,
)
svg = urllib.parse.unquote(m.group(1))
md.parseString(svg)   # raises if malformed
print(svg)
```

**b. The SVG geometry itself actually renders** — write the *raw* SVG
(decoded, not the CSS rule) to a `.svg` file and thumbnail it directly:

```bash
qlmanage -t -s 400 -o . /path/to/icon-name.svg
```

Open the resulting `.svg.png` and eyeball it. This is a real, trustworthy
render of the SVG's geometry.

**What NOT to do:** don't try to validate via a headless thumbnail of an
*HTML file* with a CSS `background-image: url(data:...)` — confirmed
unreliable (even an already-shipping icon rendered blank under `qlmanage`
run on an HTML wrapper, so a blank result there proves nothing). Beyond
that, actual in-browser visual confirmation is the user's job — ask them to
check, per their standing preference not to have UI testing improvised on
their behalf.

## 5. Sync to the installed skill copy
Ask before syncing — don't do it automatically. If the user confirms, copy
the same changed files (`assets/figma-plugin-ds.css`, `assets/demo.html`,
`references/icons.md`) over to `~/.claude/skills/plugin-ds-skill/` so the
installed skill matches. The packaged `plugin-ds-skill.skill` zip is a
separate, manual repackage step — don't touch it unless asked.

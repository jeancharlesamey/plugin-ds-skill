# Typography — making the font easy to implement

`--font-stack` is `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
Helvetica, Arial, sans-serif` — Inter first, with a native-UI-font fallback chain so
text still looks intentional even if Inter hasn't loaded yet (or never loads).

Inter itself is brought in with a single line at the very top of
`figma-plugin-ds.css` (it has to be the first rule in the file — that's a CSS rule
for `@import`, not a stylistic choice):

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap');
```

One line, no `<link>` tag needed in the HTML, no separate `@font-face` blocks to
maintain — this is the easiest way to wire up the exact three weights the tokens
were tuned for (400/500/600). That's the default this skill ships with. Two things
worth knowing before shipping it:

## It's still a network request

`@import` doesn't remove the network dependency, it just makes it a one-liner. For a
plain website that's a non-issue. **For a Figma plugin**, the plugin's UI iframe
blocks all outbound requests by default — Google Fonts included — unless the domains
are allowlisted in `manifest.json`:

```json
{
  "networkAccess": {
    "allowedDomains": ["https://fonts.googleapis.com", "https://fonts.gstatic.com"]
  }
}
```

(Both domains are needed: `fonts.googleapis.com` serves the CSS the `@import`
fetches, `fonts.gstatic.com` is where that CSS then points for the actual font
files.) Add that block once and the `@import` works unchanged inside the plugin.

## Going fully offline (no manifest entry, no network at all)

If a plugin is heading to production and should have zero network dependency for
its UI font, swap the `@import` line for local files instead:

1. Download Inter 400/500/600 as `.woff2` once (from Google Fonts or
   rsms.me/inter) and save them in a `fonts/` folder next to `figma-plugin-ds.css`.
2. Replace the `@import` line with:

```css
@font-face {
  font-family: 'Inter';
  font-weight: 400;
  font-style: normal;
  src: url('fonts/Inter-Regular.woff2') format('woff2');
}
@font-face {
  font-family: 'Inter';
  font-weight: 500;
  font-style: normal;
  src: url('fonts/Inter-Medium.woff2') format('woff2');
}
@font-face {
  font-family: 'Inter';
  font-weight: 600;
  font-style: normal;
  src: url('fonts/Inter-SemiBold.woff2') format('woff2');
}
```

`--font-stack` doesn't need to change either way — it already lists Inter first
with a solid fallback chain behind it.

## Recommendation

- Plain website / docs page / quick plugin prototype → do nothing, the `@import`
  already works.
- Figma plugin going to the Community → add the two-domain `networkAccess` block
  above (one-time, five-minute fix), or self-host if you'd rather have zero
  network dependency at all.

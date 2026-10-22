# The Field — a leadership journey

A story-driven project-leadership simulation for live YES Alumni Pakistan workshops.
Teams open one shared link, pick a codename, plan a small community project, lead it
through problems on the ground, and get a debrief.

## Run it

It is a single self-contained static page. Just open `index.html` in any modern
browser, or host the folder on any static host.

- **Entry point:** `index.html` (a copy of `the-field.html`, which is the source file).
- **Stack:** vanilla HTML + CSS + JS with inline SVG. No framework, no build step.
- **Only external dependency:** Google Fonts (Fraunces, Figtree). Everything else is inline.
- **State:** in memory only, so a page refresh restarts that device's run. One link,
  many teams, each playing independently on their own device.

## Deploy

Any static host works (Vercel, Netlify, GitHub Pages, Cloudflare Pages) with zero config,
because the root serves `index.html`.

## Editing

`the-field.html` is the working source. After changing it, refresh the deploy copy:

```
cp the-field.html index.html
```

See `the-field-CONTEXT.md` for the full build/design notes.

# MD-CMS Editor

A small browser tool that builds **tab** and **accordion** code blocks for
[MD-CMS](https://github.com/kbenestad/mdcms) — fill in items, preview them
live, then copy the generated ` ```mdcms ` block straight into a page's
Markdown source. No build step, no backend: everything runs client-side and
your draft is kept in the browser's local storage.

It shares its chrome (toolbar, header, cards, footer, theme/font-size/i18n)
with the [bizdocs](https://github.com/kbenestad/bizdocs) family of kbenestad
apps, via the self-contained **`basis`** starter template.

## What's here

```
app/
  index.html        the app — chrome + boot + block editor, all inline
  config.yml        branding + every UI string (localisation)
  help.yml          Help-modal content (kept separate from config.yml)
  assets/
    style.css       design tokens / colour scheme        ┐ shared bizdocs design
    ui.css          the kb-* UI component library         │ + runtime — bundled
    app.js          shared runtime (DOM, theme, i18n, …)  ┘ copies; DO NOT EDIT
    favicon.svg     placeholder icon — see docs/roadmap.md
    site.webmanifest
sync.sh              pull the three shared files from bizdocs
CLAUDE.md            the `basis` template this app was built from
DESIGN.md            the shared design system + pixel-perfect rules
docs/                features · unreleased · known bugs · roadmap
```

## Quick start

```bash
# serve it (the app fetches config.yml/help.yml, so file:// won't work)
cd app && python3 -m http.server 8000   # then open http://localhost:8000/
```

Pick a block type (Tabs or Accordion), a variant (Underline or Filled), add
items — each with a title, Markdown content, and whether it's active/open on
load — reorder them if needed, and copy the generated code block from the
output panel. The preview panel is interactive but simplified: it shows the
tab/accordion behaviour, not your site's exact `mdcms` theme colours.

## Staying pixel-perfect

`app/assets/style.css`, `app/assets/ui.css` and `app/assets/app.js` are
**byte-identical copies** of bizdocs' shared design + runtime. **Never
hand-edit them.** Pull bizdocs UI updates wholesale:

```bash
BIZDOCS_REF=main ./sync.sh --from-github   # from GitHub, no checkout needed
./sync.sh ../bizdocs/assets                # or from a local bizdocs checkout
git diff -- app/assets                     # review, then commit
```

See [DESIGN.md](DESIGN.md) for the full design system, and [CLAUDE.md](CLAUDE.md)
for the `basis` template this app was built from (useful if you're extending
this editor — e.g. adding another MD-CMS block type).

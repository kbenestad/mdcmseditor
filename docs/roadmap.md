# Roadmap

Where MD-CMS Editor is headed. Updated manually or by Claude Code — not tied
to any particular release.

## Long-term goal: a full MD-CMS page editor

The `Page` tab (added alongside Tabs/Accordion/Callout) is the first concrete
step: one Toast UI Editor surface for a page's Markdown body, with the same
Import/Load-from-clipboard/Validate actions as every other panel. It
deliberately does **not** yet cover:

- **Combining blocks into the page** — today Tabs/Accordion/Callout are
  separate panels with their own output; the Page tab doesn't (yet) let you
  drop a generated block into the body you're writing. See "Embedding mdcms
  blocks in the Page editor" below for why that's a bigger step than it
  sounds and a concrete plan for it.
- More block types: `toc`, `posts-created-*` (see the same reference doc),
  each as one more block kind alongside tab/accordion/callout.

### Embedding `​```mdcms​```` blocks in the Page editor as interactive widgets

Asked-for shape: while writing the Page body, dropping in a Tab/Accordion/
Callout shows up as something clickable (not raw fence text) that opens a
modal — the same item-editor form already built for those panels — and
Save writes the regenerated block back into the page.

**This is feasible, not just aspirational** — confirmed against the actual
vendored `toastui-editor-all.min.js` (3.2.2), not just the public docs:

- Toast UI Editor's underlying Markdown parser (`toastmark`) already parses
  a `​```mdcms tab-underline​```` fence as an ordinary `codeBlock` AST node,
  with `node.info` holding the fence's info string (`"mdcms tab-underline"`)
  and `node.literal` holding the body. No special parsing work needed — this
  is the same fence syntax MD-CMS itself relies on.
- `customHTMLRenderer` (confirmed present in the bundle) lets you override
  how a node type renders to the WYSIWYG DOM. A `codeBlock` override that
  checks `node.info.startsWith('mdcms')` could render an inert chip (e.g.
  "📄 Tabs · 2 items — click to edit") instead of a plain code block.
- Clicking the chip opens a `kbModal` reusing the *exact* item-editor UI this
  app already has (parse the block's YAML into the same state shape,
  `mountItemRow()` the same rows into the modal). Save regenerates the fence
  text with the same `buildSnippet()`/`buildCalloutInlineSnippet()` already
  written for the standalone panels.
- `addCommand` + `insertText`/`replaceSelection` (also confirmed present)
  cover adding a *new* block via a toolbar button, inserting a starter fence
  at the cursor.

**The open problem is identity**, not rendering: to write the edited block
back into the *right* place in the page's Markdown source, the widget needs
a stable way to say "this exact occurrence," but a page can have multiple
mdcms blocks, some potentially identical in content, and they shift position
as the user edits elsewhere. Three options, in order of how seriously worth
trying they are:

1. **An HTML comment marker** right after the closing fence (e.g.
   `<!-- mdcms-block:a1b2 -->`), assigned when the block is first created via
   the toolbar. Robust and simple to find/replace by regex. Downside: it's
   an editor-only bookkeeping artifact left in the page source — needs a
   "strip mdcms-editor markers" pass before the page is considered finished,
   and needs confirming MD-CMS's own renderer tolerates a stray comment
   between blocks (almost certainly yes — comments aren't rendered — but
   unverified).
2. **Structural position** ("the Nth mdcms code block in document order") —
   no page-source footprint, but breaks if blocks are added/reordered around
   it between opening and saving the modal. Fragile enough to rule out as
   the primary mechanism.
3. **AST node identity** (a WeakMap keyed by the node object toastmark hands
   `customHTMLRenderer`) — no footprint, but only works if toastmark reuses
   the same node object across incremental reparses for unchanged content;
   unverified without deeper experimentation, and the kind of assumption
   that's likely to quietly break on a future Toast UI version bump.

Also worth knowing going in: Toast UI Editor doesn't expose a "replace just
this node" API — regenerating one block on Save would mean a string-level
replace on the full Markdown source, then `editor.setMarkdown(newSource)`,
which resets cursor/scroll/undo history for the *whole* document. Fine for
occasional block edits; not something to build a primary editing workflow
on without checking how disruptive that feels in practice.

**Verdict:** worth building, moderate-to-high effort concentrated almost
entirely in the identity problem, not the rendering. Recommend prototyping
option 1 (HTML comment marker) first since it's the only one that doesn't
depend on an unverified assumption about Toast UI's internals.

## Long-term goal: site-aware editing (a Site tab)

Every panel today edits one block/page in isolation, with no visibility into
the rest of the site. Three related ideas want that visibility — a Site tab
that can pick a file to import, downloading a whole site, and front matter
editing with interactive sort/section/category help. Unlike the section
below, **none of this depends on any future MD-CMS core change** — it only
needs the editor to read files MD-CMS already produces today.

**The relevant files** (confirmed against `kbenestad/mdcms`'s
`docs/reference-nav.md`, `docs/reference-config.md`, `docs/reference-pages.md`,
and a real sample site under its `app/` — not guessed):

- `nav.yml` — generated by `mdcms build`, self-regenerating. `sections:`
  (code/defaultname/sort/pagesvisibility/parent/parent-sort/categorynames)
  and `pages:` (file/title/section-id/sort). Page fields are always
  overwritten from frontmatter on each build; section fields survive manual
  edits.
- `search.json` — one entry per page: file, title, section-id, keywords,
  description, author, created, modified, language, and the full markdown
  body. Draft pages are excluded from both this and `nav.yml`.
- `config.yml` — sitename/theme/PWA settings, plus
  `categories-use`/`default-category`/`categories:` (the `page.nb.md`
  language/variant system) and the `callouts:` block the Callout tab's
  Reusable mode already targets.

There is no separate "manifest" file — `nav.yml` + `search.json` together
already serve that purpose. (This supersedes an earlier manifest-file guess
floated in conversation but never previously written to this doc.)

### A Site tab: pick a file to import, or download the whole site

- Fetch `nav.yml` + `search.json` at Site-tab load to build a file picker
  (titles + sections, no per-file round trip). Selecting an entry re-fetches
  the real `pages/<file>.md` (or `posts/<file>.md`) for editing rather than
  reconstructing it from `search.json`'s extracted fields — that would be
  lossy (e.g. draft pages aren't in `search.json` at all).
- Two ways to give the editor read access to a site:
  1. **Remote fetch** — simplest, but only works if the live site sends CORS
     headers allowing the editor's origin; most static sites don't by
     default.
  2. **Local file access** (`showDirectoryPicker()`, Chromium-only) — user
     points the editor at a local checkout once; no CORS, fits this app's
     no-backend philosophy, and also makes a future write-back (regenerating
     a page in place) plausible.

  Recommend local-file-access as the primary path, remote fetch as a
  fallback for a "site URL" field.
- **Download the whole site** only really makes sense in the remote-fetch
  case — bundle every file `nav.yml`/`search.json` point at into a `.zip` via
  a CDN-loaded lib (e.g. JSZip). In the local-file-access case there's
  nothing to download; the user already has it.

### Front matter, sort, sections, categories

- ~~Front matter fields on the Page tab: title, sort, section-id, draft,
  author, created, modified, description, keywords, language — the exact key
  set in `reference-pages.md`.~~ Shipped: a config-driven Front matter card
  (`config.yml`'s `frontmatter:` block), extracted on import and prepended to
  the exported `.md`. What's left below is specifically the parts that need
  the editor to *read* the rest of the site (`nav.yml`), not the fields
  themselves.
- Sort picking needs no write-back: since `nav.yml` is fully regenerated from
  frontmatter on every `mdcms build`, the editor only needs to *read*
  `nav.yml` to show the sort values already used in the target section (so
  the user can pick one that doesn't collide) — it never needs to write
  `nav.yml` itself.
- New sections work the same way: set `section-id` in frontmatter and it
  auto-creates on the next build. Optional extra, mirroring the Callout
  tab's Reusable-mode config-snippet pattern: also generate a `nav.yml`
  `sections:` entry (defaultname/sort/parent) for the user to paste in
  immediately, if they don't want to wait for a rebuild to name/order it.
- Category scaffolding: read `config.yml`'s `categories:`/`default-category`
  list, and when creating/importing a page, offer to generate the sibling
  `page.<code>.md` stub filenames (one per configured category) with the
  same frontmatter, ready for translators to fill in.

## Dependent on MD-CMS core development

These ideas are blocked on decisions/features landing in `kbenestad/mdcms`
itself, not on anything in this repo — noted here so they aren't lost, not
scheduled.

- **Page as a block editor.** Once MD-CMS supports combining blocks into a
  page body, the Page tab should become a repeatable, ordered block list —
  like the Tabs/Accordion item editor, but heterogeneous: each entry is a
  Markdown block (Toast editor) or a Callout (reusing the existing Callout
  builder), with Tabs/Accordion joining the list too. Output is the
  concatenation of each block's generated snippet in list order. **Import
  page** would be the reverse: split a page's raw Markdown on
  `​```mdcms​```` fences (reusing the existing per-kind import parsers for
  callout/tabs/accordion) and turn the plain-text gaps between fences into
  Markdown blocks, populating the block list.
- **Steps** and **Diagrams (Mermaid)** as new block kinds, once MD-CMS's
  renderer supports them as core features.
- **Plugin support**, if MD-CMS splits into core + plugins (non-core
  features living outside the main renderer). Recommended shape: a
  declarative manifest per plugin (fence key, label, field list, an output
  template using the same `{placeholder}` token convention `config.yml`
  already uses) rather than loadable plugin JS — the editor only needs to
  know how to generate a plugin's `​```mdcms <plugin-key>​```` block, not
  how it renders on the actual site, so a thin manifest is enough. A
  `plugin-source` config setting (folder path or URL) would let the editor
  fetch an index of manifests at boot, the same way `loadYamlConfig()`/
  `loadHelpConfig()` already fetch `config.yml`/`help.yml`, and build one
  generic "plugin block" panel per manifest (reusing `field()`/`noteBox()`/
  `buildKindPanel()`, which are already fairly generic). Preview would stay
  simplified/generic, same as the existing Callout preview note. Executable
  plugin code (dynamically imported JS registered into `BLOCK_TYPES`) was
  considered and rejected for now — it means running third-party JS in the
  editor and coupling plugin authors to the editor's internal API/state
  shape, which cuts against the config-driven, no-build-step philosophy
  everything else here follows.

## Nearer-term ideas

- Drag-and-drop item reordering (currently ↑/↓ buttons only).
- Syntax highlighting in the generated-code panel (currently plain
  monospace).
- A real favicon / PWA icon set (still the `basis` template placeholder).
- Multiple in-progress drafts per block type (currently one draft each,
  autosaved to `localStorage`).
- Preview fidelity: the Preview card still renders content through the
  shared minimal `markdown()` helper, so advanced Markdown now easy to author
  in the Toast UI Editor (tables, code blocks, task lists) won't render
  correctly there even though it's valid output. Swapping in a Toast UI
  Editor read-only `Viewer` per item would fix this, but needs a debounced
  rebuild (not per-keystroke) to avoid the cost/churn of creating a Viewer
  instance on every keystroke — deferred until it's worth the complexity.
- Callout reusable-message entries don't have the fold/collapse treatment
  Tab/Accordion items got — skipped for now since these lists are typically
  short (a handful of language variants), but the same `mce-fold-btn`
  pattern would drop in cleanly if that stops being true.
- Validate currently re-parses/re-checks structure on demand (button click);
  it could run continuously and surface issues inline instead of behind a
  modal, once it's clear that wouldn't just be noise while mid-edit.

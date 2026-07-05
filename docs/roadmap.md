# Roadmap

Where MD-CMS Editor is headed. Updated manually or by Claude Code — not tied
to any particular release.

## Long-term goal: a full MD-CMS page editor

The `Page` tab (added alongside Tabs/Accordion/Callout) is the first concrete
step: one Toast UI Editor surface for a page's Markdown body, with the same
Import/Load-from-clipboard/Validate actions as every other panel. It
deliberately does **not** yet cover:

- **Front matter** (`title`, `sort`, `section-id`, `draft`, `created`, …) —
  see `kbenestad/mdcms`'s `docs/reference-pages.md` for the full key set. The
  natural next step once the body editor itself is solid.
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

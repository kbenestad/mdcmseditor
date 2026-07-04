# Roadmap

Where MD-CMS Editor is headed. Updated manually or by Claude Code — not tied
to any particular release.

## Long-term goal: a full MD-CMS page editor

Tabs and accordions are the first two block types; the editor's data model
(`BLOCK_TYPES` in `index.html`) is deliberately generic so this can grow into
editing an entire page's Markdown — front matter, prose, and every MD-CMS tag
— without a rewrite. Concretely, that means:

- More block types: `callout-*`, `toc`, `posts-created-*` (see
  `kbenestad/mdcms`'s `docs/reference-pages.md`), each as one more
  `BLOCK_TYPES` entry + a per-kind panel builder.
- Editing/importing an existing `.md` file (front matter + body), not just
  generating one isolated block at a time.
- A combined output: build a whole page from multiple blocks in one document,
  not one snippet per block-type panel.

## Nearer-term ideas

- Drag-and-drop item reordering (currently ↑/↓ buttons only).
- Syntax highlighting in the generated-code panel (currently plain
  monospace).
- Import an existing `​```mdcms …​```` block to edit it, instead of only
  building one from scratch.
- A real favicon / PWA icon set (still the `basis` template placeholder).
- Multiple in-progress drafts per block type (currently one draft each for
  Tabs and Accordion, autosaved to `localStorage`).

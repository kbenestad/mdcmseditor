# Unreleased

Features that have landed on `claude/mdcms-editor-webapp-3g99aq` but haven't
shipped to `main` yet.

Once a PR is pushed, update this file to reflect reality: move shipped items
out (they belong in [`features.md`](features.md)) and keep only what's still
unreleased.

- The MD-CMS Editor itself: a block-type switch (Tabs / Accordion), each with
  a variant toggle (Underline / Filled), a repeatable item editor (title,
  Markdown content, title-style, active/open-on-load), reordering, a live
  interactive preview, and a generated `​```mdcms …​```` code block with
  Copy-to-clipboard and Download-as-.md actions.
- Draft autosave to `localStorage` (`mdcmseditor-draft`) — a reload keeps the
  in-progress items.
- Config trimmed to what this app actually uses: dropped the PDF-only header
  keys (`page-size`, `output-language`, fonts, `logo`) since this app never
  produces a PDF.
- `pdf-lib` removed from the CDN block for the same reason — see the
  file-header comment in `index.html`.
- Item content is now edited with Toast UI Editor (WYSIWYG, pinned 3.2.2) —
  real rich-text editing with a one-click switch to raw Markdown mode, not
  just a formatting toolbar over plain text. Dark mode is kept in sync with
  the app's own theme toggle. Image insertion prompts for a URL/path instead
  of embedding a base64 blob into the generated YAML.
- Each item can be folded/collapsed independently (a compact header showing
  just its title), plus a card-level "Collapse all" / "Expand all" control —
  keeps long item lists manageable.
- Item rows are now mounted once and patched in place (add/remove/reorder/fold
  no longer trigger a full page rebuild) so Toast UI Editor instances aren't
  destroyed and recreated on unrelated actions — see the "Item row lifecycle"
  note at the top of `index.html`.

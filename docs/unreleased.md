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

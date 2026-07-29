# Features

The running list of what MD-CMS Editor provides, kept in sync with what's
actually shipped on `main`.

- Four editing panels, switched via tabs: **Page**, **Tabs**, **Accordion**,
  **Callout**. Each has its own Preview and Generated-code cards.
- **Page**: a single Markdown body for a whole page, plus a config-driven
  **Front matter** card on top — extracted from an imported file, edited with
  the right widget per field (text, number, boolean, dropdown, date,
  date-time with an optional time and a one-click "Now" button), and
  prepended to the exported `.md` as a YAML block. Field keys/types/labels are
  defined in `config.yml`'s `frontmatter:` block; a key imported from a file
  but not listed there still round-trips via an "Additional keys" list, which
  also lets you add an ad hoc key by hand. (Combining blocks into the body is
  not yet supported — see `docs/roadmap.md`.)
- **Tabs** / **Accordion**: a variant toggle (Underline / Filled), a
  repeatable item editor (title, Markdown content, title-style, active/
  open-on-load), reordering, per-item fold/collapse plus a card-level
  "Collapse all" / "Expand all" control, and a live interactive preview.
  Accordion's preview allows any number of items open simultaneously
  (matching real MD-CMS behaviour), unlike the shared single-open
  `makeAccordion()` helper.
- **Callout**: two modes — **Inline** (type + optional title/icon + body →
  one `​```mdcms callout-*​```` block) and **Reusable** (a message key + type +
  per-category-code title/text entries → both the page-side
  `​```mdcms callout-*\nmessage: <key>​```` snippet *and* the corresponding
  `callouts:` entry to paste into config.yml).
- Every panel: **Import .md** (file picker), **Load from clipboard**, and
  **Validate** (structural checks — missing fields, duplicate titles/codes,
  unbalanced code fences, a YAML round-trip check — shown in a modal).
  Import/clipboard recognise this app's own generated blocks, so existing
  `​```mdcms​```` blocks (or, for callouts, an existing `callouts:` config
  entry) can be brought back in and edited.
- Item/page/callout content is edited with Toast UI Editor (WYSIWYG, pinned
  3.2.2) — real rich-text editing with a one-click switch to raw Markdown
  mode. Dark mode stays synced with the app's own theme toggle. Image
  insertion prompts for a URL/path instead of embedding a base64 blob.
  Pasted/typed content is sanitised (`sanitizeMarkdown()`) so characters like
  non-breaking spaces or a BOM can't silently break the generated YAML's
  `content: |` block style into an escaped, double-quoted string.
- Copy-to-clipboard and Download-as-.md actions on every generated code
  block.
- Draft autosave to `localStorage` (`mdcmseditor-draft`) — a reload keeps
  all in-progress panels (including fold state).
- Item/entry rows are mounted once and patched in place — add/remove/
  reorder/fold, mode switches, and imports never trigger a full page
  rebuild, so Toast UI Editor instances aren't destroyed and recreated on
  unrelated actions (see the "Item row lifecycle" note at the top of
  `index.html`).
- Config trimmed to what this app actually uses: no PDF-only header keys
  (`page-size`, `output-language`, fonts, `logo`) and no `pdf-lib` — this
  app's output is text, never a PDF.
- Shared bizdocs design system, bundled locally in `assets/` (`style.css`,
  `ui.css`, `app.js`) and kept byte-identical via `sync.sh`.
- Config-driven branding and localisation (English/German) via `config.yml`,
  validated at boot by `assertValidConfig()` so a missing/misserved config
  fails loudly instead of silently rendering broken chrome.
- Theme (light/dark) and font-scale controls, persisted to `localStorage`.

# Features

The running list of what `basis` provides, kept in sync with what's shipped
on `main`.

- Single-file app: chrome, boot, form, and PDF generation all inline in
  `index.html` — no build step, no backend.
- Shared bizdocs design system, bundled locally in `assets/` (`style.css`,
  `ui.css`, `app.js`) and kept byte-identical via `sync.sh`.
- Config-driven branding and localisation via `config.yml`, validated at boot
  by `assertValidConfig()` so a missing/misserved config fails loudly instead
  of silently rendering broken chrome.
- Theme (light/dark) and font-scale controls, persisted to `localStorage`.
- PDF output via `pdf-lib`.
- Example form (`buildExampleCard()`) as a starting point for a real app form.

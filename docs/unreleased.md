# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

Once a PR is pushed, update this file to reflect reality: move shipped items
out (they belong in [`features.md`](features.md)) and keep only what's still
unreleased.

- Icons (Google Material Symbols Outlined, fetched at boot from
  `assets/icons/*.svg`) on a deliberately short, specific list of spots:
  Import .md, Load from clipboard, Validate, the Callout Inline/Reusable mode
  buttons, and the five card titles (Page content, Items, Callout, Preview,
  Generated code). Nothing else in the app got an icon.
- A right-aligned **Help** button in the toolbar, opening a modal with brief,
  non-technical getting-started guidance — including how to bring an
  existing `​```mdcms​```` block (or `callouts:` config entry) back in via
  Import .md / Load from clipboard to fix it.

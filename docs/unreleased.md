# Unreleased

Features that have landed on `development` but haven't shipped to `main` yet.

Once a PR is pushed, update this file to reflect reality: move shipped items
out (they belong in [`features.md`](features.md)) and keep only what's still
unreleased.

- **Front matter** on the Page tab: a config-driven card (see `config.yml`'s
  `frontmatter:` block) extracted on import and prepended to the exported
  `.md`. Field types: text, number, boolean, dropdown, date, and date-time
  (time optional, dropped from the output entirely when not set; a "Now"
  button can seed either date or date+time per field). A new page starts with
  the config's `default:` values seeded (e.g. `sort: 100`); importing a file
  with no frontmatter block resets to fully blank instead, so re-exporting an
  untouched import never fabricates a block the file didn't have. Keys not
  listed in `config.yml` still round-trip through an "Additional keys" list,
  which also lets you add one by hand. Parses with `jsyaml.load(...,
  { schema: jsyaml.CORE_SCHEMA })` and emits date/number/boolean lines by
  hand (not `jsyaml.dump()`) — see the file-header note in `index.html` for
  the two verified js-yaml quirks this avoids.
- Icons (Google Material Symbols Outlined, fetched at boot from
  `assets/icons/*.svg`) on a deliberately short, specific list of spots:
  Import .md, Load from clipboard, Validate, the Callout Inline/Reusable mode
  buttons, and the five card titles (Page content, Items, Callout, Preview,
  Generated code). Nothing else in the app got an icon.
- A right-aligned **Help** button in the toolbar, opening a modal with brief,
  non-technical getting-started guidance — including how to bring an
  existing `​```mdcms​```` block (or `callouts:` config entry) back in via
  Import .md / Load from clipboard to fix it.
- All runtime app files (`index.html`, `config.yml`, `assets/`) moved into an
  `app/` subdirectory, so the whole deployable unit is one directory; process
  docs (`CLAUDE.md`, `DESIGN.md`, `README.md`, `docs/`, `sync.sh`) stay at the
  repo root. Help-modal content moved out of `config.yml` into its own
  `app/help.yml`, fetched at boot alongside `config.yml`, so it can be
  edited/reviewed independently of branding and UI strings.

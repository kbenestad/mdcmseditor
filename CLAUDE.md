# CLAUDE.md

Guidance for building a **standalone kbenestad business-document app** from this
template. For the design system and the rules that keep an app pixel-identical to
the bizdocs family, see [DESIGN.md](DESIGN.md).

## What this is

This repository is **`basis`** — a self-contained starter for a single small web
app that produces a business document (invoice, quote, packing slip, contract,
…) as a PDF, looking and behaving **exactly** like the apps in the
[bizdocs](https://github.com/kbenestad/bizdocs) series, but shipping on its own
(its own repo, its own hosting), with **no build step and no backend**.

The whole app is **one `index.html`** (all HTML/CSS/JS inline) plus a
`config.yml` and a bundled `assets/` folder — all of it under `app/`, so the
whole deployable unit is one directory. You build an app by copying this
template and replacing the marked app-specific parts — never by hand-assembling
the chrome, which is what guarantees the pixel-perfect match.

## Repository layout

```
app/
  index.html        the app: chrome + boot + your form + your PDF (all inline)
  config.yml        branding + all UI strings (localisation), read at boot
  help.yml          Help-modal content (kept separate from config.yml so it
                    can be edited/reviewed independently of branding/strings)
  assets/
    style.css       design tokens / colour scheme + reset + page shell   ┐ shared
    ui.css          the kb-* UI component library                        │ design,
    app.js          shared runtime (DOM, theme, modals, i18n, config, …) ┘ DO NOT EDIT
    favicon.svg     placeholder icon — replace with a real set per app
    site.webmanifest
sync.sh              refresh the three shared files from bizdocs (see below)
CLAUDE.md · DESIGN.md · README.md
```

### The shared files are bundled, and must stay byte-identical

Unlike an in-series bizdocs app (which links one shared `../assets/`), this app
**bundles its own copy** of the design + runtime in `app/assets/` and
references it locally (`assets/style.css`, not `../assets/style.css`, since
`index.html` is a sibling in `app/`). That is what makes the folder standalone.

The price of self-containment is duplication, so the discipline is strict:
**`app/assets/style.css`, `app/assets/ui.css` and `app/assets/app.js` are
byte-identical copies of bizdocs' shared files. Never hand-edit them.** A
bizdocs UI change is pulled in wholesale, not patched here:

```bash
BIZDOCS_REF=main ./sync.sh --from-github   # pull the three shared files from GitHub
# or, if a bizdocs checkout sits alongside this repo:
./sync.sh ../bizdocs/assets
git diff -- app/assets                     # review, then commit
```

If you ever need app-specific CSS, it goes in `index.html`'s inline `<style>`;
if you need app-specific copy, it goes through `config.yml` + `S()`. The moment
you edit the shared files directly, drop-in sync stops being clean and the app
drifts from the family. Don't.

### A bug *in* the shared layer is not fixed here — or in appdevelopment

If something in `style.css`, `ui.css`, or `app.js` is actually broken — not
"this app needs different behaviour" but "this component/rule is wrong for
everyone who uses it" — the fix does **not** belong in this app's inline
`<style>`/`<script>` as a local override or workaround, and it does **not**
belong hand-patched into this repo's bundled `app/assets/` copies either.
Either one only fixes the symptom in this one app; every other app in the
family, and the template itself, keeps shipping the bug.

Fix it at the true, canonical source, always:
**[kbenestad/bizdocs](https://github.com/kbenestad/bizdocs)** `assets/` —
pushed straight to its `main` branch (bizdocs has no `development` branch;
see its own CLAUDE.md). That is the *one* original copy of
`style.css`/`ui.css`/`app.js`. Every basis-template app — including
[kbenestad/appdevelopment](https://github.com/kbenestad/appdevelopment), the
private repo this app's own template was copied from — bundles a
byte-identical *copy* of bizdocs' files and is downstream of it. Fixing the
bug only in appdevelopment is not enough: appdevelopment's `assets/` is
itself just another synced copy, not the original, and the next sync there
would silently overwrite a hand-patch anyway.

Once the fix lands in bizdocs `main`:
1. In `appdevelopment` (on its `development` branch), run its `sync.sh` to
   pull the fix in — and if the bug is in a component appdevelopment's own
   template doesn't yet exercise, add the minimal demo code needed to cover
   it, so future apps copied from the template inherit a working reference.
2. In **this** repo, run `sync.sh` to pull the same fix in here.

Only genuinely app-specific behaviour gets an inline override in this repo;
a shared bug gets fixed exactly once, in bizdocs, and flows downhill to
every app in the family via `sync.sh`.

## How the app boots

1. A tiny inline pre-paint script in `<head>` reads `localStorage['kb-theme']`
   and sets `data-theme` before first paint (avoids a flash). Keep it.
2. CDN libraries load: `js-yaml` (config) and `pdf-lib` (PDF output).
3. `assets/app.js` (i.e. `app/assets/app.js`, fetched relative to `index.html`)
   loads and defines the shared globals.
4. The inline `<script>` runs: `loadYamlConfig()` fetches + parses `config.yml`,
   **`assertValidConfig()` validates it** (see below), `normaliseConfig()`
   flattens the `localisation:` block, `applyAccent()` / `initFontScale()` /
   `applyLang()` apply settings, and `render()` builds the UI. State persists to
   `localStorage` under app-specific keys.

Boot order to preserve: `loadYamlConfig → assertValidConfig → normaliseConfig →
applyAccent → initFontScale → applyLang → render`.

### Config validation (keep this guard)

`config.yml` is fetched at runtime. On a static host a **missing** `config.yml`
— or an SPA fallback, or a stale cached page — is served as an **HTML page with
a 200**. `jsyaml.load()` then returns a plain *string*, `CFG.localisation` is
undefined, `normaliseConfig()` silently early-returns, and the app paints its
full chrome with **every label showing as a raw key, an empty language dropdown,
and no error at all**. It looks broken with no clue why.

`assertValidConfig()` in the inline boot turns that into a clear, actionable
error:

```js
function assertValidConfig(cfg) {
  if (!cfg || typeof cfg !== 'object' || !cfg.localisation)
    throw new Error('config.yml loaded but did not parse to a valid configuration. '
      + 'The server most likely returned an HTML page (a 404 / SPA fallback, or a '
      + 'stale cached page) instead of the YAML file. …');
}
```

Keep this in the inline boot — **not** in `assets/app.js`, which must stay
byte-identical to bizdocs. (`loadYamlConfig()` deliberately doesn't validate
shape; that's the app's job.)

## Building your app from this template

1. **Rename / rebrand.** Set the `<title>`, the doc-title `<h1>`, the brand
   fallback text, and the `basis-lang` localStorage key (and any other per-app
   keys) to your app's name.
2. **Build the form.** Replace `buildExampleCard()` with your real form, built
   **only from `kb-*` components** (see DESIGN.md). Put any bespoke layout
   (column grids, repeating rows, signature pad) in the inline `<style>`.
3. **Build the PDF.** Replace `onDownload()` with your real document using
   **pdf-lib** (`PDFLib`, already loaded).
4. **Fill in `config.yml`.** Keep the header keys; add every user-facing string
   to the `ui:` block of **every** language; route all copy through `S('key')`.
5. **Drop in real icons.** Replace `assets/favicon.svg` / `site.webmanifest`
   with a full favicon / PWA icon set.

## Running / previewing locally

The app `fetch`es `config.yml` (and `help.yml`), so it must be served over
HTTP — opening `index.html` via `file://` will fail.

```bash
cd app && python3 -m http.server 8000
# then open http://localhost:8000/
```

## Verifying a change

There are no automated tests — verify visually by rendering the app with a
headless Chromium:

```bash
CHROME=/path/to/chromium      # e.g. /opt/pw-browsers/chromium-*/chrome-linux/chrome
"$CHROME" --headless=new --no-sandbox --disable-gpu \
  --virtual-time-budget=8000 --window-size=1200,1600 \
  --screenshot=out.png "http://localhost:8000/index.html"
```

(serve from inside `app/`, as above, so the URL is `.../index.html` with no
`app/` prefix.) `--virtual-time-budget` lets the JS-built UI settle before the
screenshot.

**Caveat:** in sandboxed environments the browser often cannot reach the CDNs,
so `js-yaml` fails to load and you'll see the config error. To verify a full
render, vendor `js-yaml` locally **for the test only** — download it, drop a copy
into `app/assets/`, point a throwaway copy of `index.html` at the local file, and
screenshot that. Delete the throwaway files afterwards; never commit them.
(`pdf-lib` is only needed to generate a PDF, not for the initial render.) Worth
screenshotting after a change: the full form, dark mode, the About modal, and a
non-English language.

## Development workflow

Day-to-day development happens on the **`development`** branch, not `main`.
Before a push to `main`, open a pull request — **the user decides when a PR is
opened**, don't push to `main` unopenedly on your own initiative.

**There are exactly two branches for this repo: `development` and `main`.
Never use, invent, or leave work stranded on any other branch — no
`claude/whatever-slug`, no `fix/this-bug`, no per-task branch of any kind,
ever, for any reason.** A task-scoped branch fragments history, makes
`docs/unreleased.md` inaccurate the moment a second task branches off the
same tip, and leaves a stray ref nobody cleans up. This has happened
repeatedly (e.g. `claude/add-icons-help-modal`, `claude/nav-line-overflow-…`)
and every instance of it is a mistake to not repeat, not a precedent to
follow.

This rule applies **even when a session's own harness/runtime hands you a
different branch name as that session's default** (Claude Code on the web
does this routinely). That default is a mechanism of the *session* — it is
not a decision about where this app's history lives, and it is never a
license to commit there and call the work done. When that happens:

1. Do the work and commit it — you may have no choice about the harness
   making its own branch current.
2. Before finishing, create `development` if it doesn't already exist
   (branch it from `main`, or from the harness branch's tip if that tip
   has newer work `main` doesn't), and push your commits there too — via
   fast-forward, merge, or a fresh branch-and-push of the same tip, whichever
   applies. `development` must end up with every commit, in the same order.
3. Treat the task as unfinished until `development` on the remote actually
   has the work. A commit that exists only on a throwaway harness branch —
   however "pushed" it looks — is not done.

If you are ever unsure whether this rule was actually satisfied, run
`git ls-remote --heads origin` and check `development` is there and current
— don't assume from having pushed *somewhere*.

While on `development`, keep these docs current:

- **`docs/unreleased.md`** — features that have landed on `development` but
  haven't shipped to `main` yet. Once a PR is pushed, update this doc so it
  reflects reality (move shipped items out, keep only what's still unreleased).
- **`docs/features.md`** — the running list of features the app has, kept in
  sync with what's actually shipped on `main`.
- **`docs/known-bugs.md`** — known bugs. Remove an entry the moment its bug is
  fixed; this file should only ever list what's currently broken.
- **`docs/roadmap.md`** (or `docs/roadmap.html`) — where the app is headed.
  Updated manually or by Claude Code, not tied to any particular release.

## Versioning (`VERSION.md`)

`VERSION.md` in the repo root tracks the app's release version, in this
format:

```
**kbenestad/{reponame} • {App name}**
**Version:** X.Y.Z - latest commit {commit}
**Last updated:** {Date - format d Mmmm YYYY}
```

Update it whenever a push to `main` happens:

- **X (major)** — a defined feature set is complete, or a breaking change
  makes the app incompatible with previous versions.
- **Y (minor)** — new features ship, increasing what the app can do.
- **Z (point)** — a new iteration ships, typically one PR. Several PRs can
  land before a point release if they're part of the same iteration — ask the
  user if it's unclear whether a batch of PRs warrants its own point release.

## Footer

Every app built from this template must have a footer. Right-aligned, two
lines:

```
vX.Y.Z • {commit}
{d Mmmm YYYY}
```

The version/commit line matches `VERSION.md`; the second line is its
"Last updated" date. The commit links to the latest PR (the PR that shipped
it), not to a raw commit view.

Each PR description must:

- Mention any commits that landed directly on `main` without their own PR
  since the previous PR.
- Link back to the previous PR.

## Conventions (the pixel-perfect rules)

- **Reuse the shared layer; never fork it.** Styling and cross-cutting logic
  live in `assets/`. Don't reintroduce design tokens, `kb-*` components, DOM
  helpers, or theme/modal/i18n/format code in the app — use what `app.js` and
  the CSS already provide.
- **Same classes/IDs for the same element** as the bizdocs apps, so a synced
  `ui.css` change lands correctly. Only genuinely app-specific layout belongs in
  the inline `<style>`.
- **Scope.** The main inline script is wrapped in an IIFE
  (`(async function(){ … })()`), so its functions are not globals; the globals
  from `app.js` ($, el, makeSizeControl, kbAbout, lookupString, …) are visible
  inside it.
- **localStorage keys.** Theme = `kb-theme`, font scale = `kb-font-scale`
  (shared, don't rename). Per-app data uses your own keys (e.g. `basis-lang`).
- **No hardcoded user-facing text.** Add a key to every language block in
  `config.yml` and look it up via `S()`. Use `{placeholder}` tokens for values.
- **PDF output uses pdf-lib.** It can draw from scratch and embed/append
  existing PDF/image bytes (receipts, signatures), so it covers every app.
- **The container is ephemeral / hosting is static.** Commit your work; deploy
  the whole `app/` directory (`index.html` + `config.yml` + `help.yml` +
  `assets/`) together as plain static files.

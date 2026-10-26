# htmx

htmx is a dependency-free browser library that adds AJAX, CSS transitions, WebSockets and
SSE to plain HTML through `hx-*` attributes. This checkout is **htmx 2.x** (`master`,
version in `src/htmx.js` and `package.json`). Upstream also has `dev` (where code PRs go)
and `four`/`four-dev` (htmx 4, a different API): docs or answers about htmx 1 or 4 do not
apply here.

## Where things are

- `src/htmx.js`: **the entire library, one file** (~5,400 lines, an IIFE assigned to
  `var htmx`). The public API is the `htmx` object at the top; `htmx.config` defaults are
  just below it. Read attributes through `getAttributeValue` (it also accepts the
  `data-hx-*` form) or `getClosestAttributeValue` (adds inheritance), never
  `elt.getAttribute`, or the `data-` variant silently stops working.
- `test/attributes/hx-<name>.js`: one test file per attribute. `test/core/`: everything
  else (`api.js`, `events.js`, `regressions.js`, `security.js`, ...). Only `.js` files in
  these two directories are run. `test/manual/` is for by-hand browser checks, not CI.
- `www/`: the htmx.org website (Zola). Attribute docs are `www/content/attributes/hx-*.md`;
  the summary table is `www/content/reference.md`.

## Commands

```bash
npm ci                                                  # install (Node 20+, as CI uses)
npm test                                                # lint + type check + headless Chromium, ~40s
npm test test/attributes/hx-swap.js                     # one file; lint and type check still run on everything
npx web-test-runner --browsers chromium --playwright test/attributes/hx-swap.js   # one file, ~2s, no lint
npm run format                                          # eslint --fix; run before finishing
npm run types-check                                     # tsc --checkJs on src/htmx.js
```

**Done means `npm run format && npm test` both pass.** The one-file command is for
iterating; it skips lint and the type check and runs one file of fifty.

`npm test` installs Chromium through Playwright itself; `CHROME_PATH` (mentioned in
CONTRIBUTING.md) is not needed. A clean checkout passes: 868 passed, 3 skipped.

## Conventions

- Tests: mocha + chai `should` + a sinon fake server. Copy the shape of the existing file:
  `beforeEach` sets `this.server = makeServer()` and calls `clearWorkArea()`; build DOM with
  `make('<div hx-get="/test">')`; stub with `this.server.respondWith('GET', '/test', '...')`;
  trigger (`div.click()`), then `this.server.respond()`, then assert. Use `function()` for
  `describe`/`it`/hooks, never arrow functions, because tests rely on `this.server`.
  Helpers (`make`, `byId`, `clearWorkArea`, `makeServer`) are in `test/util/util.js`.
- Style is eslint `standard` with overrides in `package.json`: no semicolons, 2 spaces, no
  space before function parens (`function(a)`), `.slice`/`.substring` not `.substr`.
- New or changed functions in `src/htmx.js` need JSDoc types: `types-check` runs `tsc
  --checkJs` against it and fails the build.
- A behaviour change touches three places: `src/htmx.js`, a test in `test/`, and the doc
  page in `www/content/`.
- Commit subjects are short and imperative ("fix hx-push-url false on boosted links"),
  with no `[scope]` prefix.

## Do not edit

- `dist/`: built from `src/` by `npm run dist` at release time. Tests load `src/htmx.js`,
  so a change in `dist/` is never tested. `dist/ext/*` are deprecated copies; extensions
  now live in the separate `bigskysoftware/htmx-extensions` repo.
- `www/static/src/`, `www/static/test/` and `www/themes/htmx-theme/static/js/htmx.js`:
  stale copies made by `npm run www`. Searching the repo for a function returns **seven**
  copies of the source; only `src/htmx.js` is real. Exclude `dist/` and `www/` when grepping
  for code.
- `editors/jetbrains/htmx.web-types.json`: generated from `www/content/attributes/` by
  `npm run web-types-generate`.
- Do not run `npm run dist` or `npm run www` unless asked; they rewrite tracked files
  across `dist/` and `www/`. `npm run update-sha` is broken (its script does not exist).

# Week 4 Write-up

**Repo chosen** and link to your fork:
> [bigskysoftware/htmx](https://github.com/bigskysoftware/htmx), the htmx 2.x line
> (`master`, version 2.0.11). Fork: https://github.com/zubincarvalho/htmx

**File you wrote** (`AGENTS.md` or `CLAUDE.md`):
> `CLAUDE.md`, at the root of the fork, with a copy in `week4/CLAUDE.md`. I chose
> `CLAUDE.md` because Part III was tested with Claude Code, which loads that file
> automatically, so there is no doubt the agent saw it during the trials.


## Part I: Exploration

| | Finding |
|---|---|
| Build / run | No build step for development: tests and the website load `src/htmx.js` directly. `npm run dist` (`scripts/dist.sh`) copies it into `dist/` as IIFE, AMD, CJS and ESM builds plus a minified and gzipped copy, and is only run at release. There is nothing to "run" locally besides the tests; the website is Zola (`zola build` in `www/`, per `netlify.toml`). |
| Test | `npm ci`, then `npm test` (lint, then `tsc` type check, then the browser suite in headless Chromium through web-test-runner and Playwright). Clean checkout: 50 files, 868 passed, 3 skipped, about 40 seconds. One file: `npm test test/attributes/hx-swap.js` works but still lints and type-checks everything (about 4 s); `npx web-test-runner --browsers chromium --playwright <file>` runs only that file (about 1.5 s). |
| Lint / format | `npm run lint` (eslint `standard` with overrides in `package.json`), `npm run format` (`eslint --fix`), `npm run types-check` (`tsc --checkJs` on `src/htmx.js`). All three are part of `npm test`. |
| Key directories and entry points | `src/htmx.js` is the whole library: one 5,400-line file, an IIFE assigned to `var htmx`, with the public API object and `htmx.config` defaults at the top. `test/attributes/` has one test file per `hx-*` attribute; `test/core/` has the rest. `www/` is the htmx.org website, with attribute docs in `www/content/attributes/`. `web-test-runner.config.mjs` holds the test page boilerplate and the file globs. |
| Unstated conventions | Tests always have the same shape: `this.server = makeServer()` (a sinon fake server) and `clearWorkArea()` in `beforeEach`, `make('<div hx-get=...>')`, `this.server.respondWith(...)`, a click, `this.server.respond()`, then a chai `should` assertion. They use `function()` rather than arrows (480 `this.server.respond()` calls depend on `this`). Attributes are read through `getAttributeValue`, which also accepts the `data-hx-*` form; only two raw `getAttribute` calls exist in the source. New functions need JSDoc types because of `tsc --checkJs`. Commit subjects are short and imperative, with no scope prefix. |
| Off-limits (generated, vendored) | `dist/` (generated from `src/`; `dist/ext/` is deprecated, since extensions moved to a separate repo). `www/static/src/`, `www/static/test/` and `www/themes/htmx-theme/static/js/htmx.js` are copies made by `npm run www`, and they are stale: `diff` against `test/` shows missing and differing files. `editors/jetbrains/htmx.web-types.json` is generated from the attribute docs. |

**How you figured it out**: what you asked the agent, what you had to read yourself:
> I used an agent (Claude Code) for the exploration, starting with the read-only survey
> script from my week 3 skill (`week3/where-to-start/scripts/survey.sh`). It listed the
> docs, the single CI workflow, the layout by file count, and the recent commit style in
> under a second. From there the agent read `package.json`, `CONTRIBUTING.md`,
> `TESTING.md`, the CI workflow, `scripts/` and `web-test-runner.config.mjs`, and ran
> every command in the table above rather than trusting the docs.
>
> The findings that mattered most came from running and diffing things, not from reading:
>
> - **Seven copies of the source.** Searching for `function getClosestAttributeValue`
>   returned seven files: `src/`, four in `dist/`, and two under `www/`. Only `src/` is
>   loaded by the tests. Nothing in the repo says so.
> - **Stale website copies.** `diff -rq test www/static/test` showed the website's copy of
>   the tests missing whole files, so those copies are not just generated but out of date.
> - **Stale or broken docs.** `CONTRIBUTING.md` says `CHROME_PATH` must be set; the suite
>   passed with it unset, because the test script installs Chromium itself.
>   `npm run update-sha` points at `scripts/update-sha.sh`, which does not exist.
> - **The single-file command.** `TESTING.md` gives `npm test <file>`, and running it
>   showed npm appends the argument only to the last script, so lint and the type check
>   still run on the whole repo.
> - **Other branches.** `git ls-remote` against upstream showed `dev` (where code PRs go)
>   and `four`/`four-dev` (htmx 4). My fork copied only `master`, which is htmx 2.
>
> One claim I had to check and drop: I first listed `src/htmx.esm.d.ts` as generated.
> Its history shows it was restored by hand ("restore ts file"), and I could not confirm
> what produces it, so it is not in the file.


## Part II: The File

**Anything you deliberately left out**, and why:
> - **What htmx is for, at length.** The README and the website do that. The file has
>   one sentence of domain, plus the version line, because the version is what an agent
>   gets wrong.
> - **The 36 attribute pages and the event list.** They are documented in `www/content/`, and
>   an agent can read the one it needs. Listing them would roughly double the file.
> - **Firefox and WebKit commands** (`npm run test:firefox`, `test:webkit`, `test:all`).
>   CI runs only `npm test`, which is Chromium; the rest download extra browsers and are
>   for maintainers chasing browser-specific bugs.
> - **`test/manual/` and `npm run test:debug`.** Both need a person watching a browser.
>   The file says only that `test/manual/` is not part of CI, so an agent does not put
>   tests there.
> - **Contribution process** (PRs go to `dev`, issues before features, refactors get
>   closed). It is in `CONTRIBUTING.md`, it is about humans submitting PRs, and an agent
>   working in a fork does not need it. The file keeps only the one consequence that
>   matters for code: htmx 4 on other branches is a different API.
> - **Generic advice** such as "write tests" or "keep changes small". A competent model
>   already does that.

**The hardest call** on what to include:
> Whether to give the slow single-file command or the fast one. `TESTING.md` documents
> `npm test <file>`, which is what a maintainer would expect to see. But I measured that
> it re-lints and re-type-checks the whole repo every time, while calling web-test-runner
> directly runs the one file in about a second and a half. Documenting only the fast one
> risked an agent never running lint at all; documenting only the documented one wastes
> time on every iteration. I kept both, each with a comment saying what it skips. Part III
> showed that was not enough on its own (below).


## Part III: Trying It Out

Both tasks were run in fresh, headless Claude Code sessions (`claude -p`) in the fork's
root, with `CLAUDE.md` in place and nothing else said about the repo. Sessions could
read, search, run commands and edit files; `git push` was blocked.

| # | Task | Did the agent use your commands and conventions? |
|---|---|---|
| 1 | "Run the test suite and tell me whether anything fails." | Yes. Its only command was `npm test`, and it reported 868 passed, 0 failed, 3 skipped. It also compared the result against the clean-checkout numbers in `CLAUDE.md`, and correctly said the 404/400 messages in the browser log come from tests of error handling. Two turns. |
| 2 | "Small change: make sure hx-confirm also works when it's written as data-hx-confirm, and add a test for it." | Mostly. It searched `src/htmx.js` only, so the seven-copies trap never came up, and it found that `getAttributeValue` already accepts `data-hx-confirm`. It added two tests in exactly the repo's idiom (`function()`, `make`, `respondWith`, `respond`, `should`), did not touch `dist/`, and correctly made no docs change, since behaviour did not change. **But** it verified with the fast one-file command and `npx eslint` on that one file, never ran `npm run format`, and never ran the full `npm test`. |

**One thing you'd change** about the file after watching it get used:
> **State a definition of done.** In task 2 the agent picked the fastest command in the
> file and stopped there, because nothing said when a change counts as finished. "Run
> before finishing", as a comment on the `format` line, was not enough. I added one line
> under the commands:
>
> > **Done means `npm run format && npm test` both pass.** The one-file command is for
> > iterating; it skips lint and the type check and runs one file of fifty.
>
> Then I reverted the test file and re-ran task 2, word for word, in a new session. This
> time it finished with `npm run format && npm test` and reported 870 passed (the 868
> baseline plus its two new tests). I reverted the test file again afterwards, so the
> fork contains only `CLAUDE.md`.


## Submission
1. `Command (⌘) + F` for `TODO`. No results means you're done.
2. Push all changes to your remote repository and submit via Gradescope.

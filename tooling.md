# Build, test and release tooling

Source plugins: GCC, O2T, OVC (see the README). File paths are in the named
plugin's repository.

## Architecture that makes testing possible

- `src/core/` pure (no Obsidian, Electron or Node imports, enforced by
  ESLint), `src/plugin/` holds the side effects, `main.ts` only wires
  (SCAR-031). OVC: an in-memory compiler over a `files` map.
- Shared logic between export and preview lives once (SCAR-017).

## Build and bundling

- **esbuild**, CJS, platform node, target es2022. Externals: `obsidian`,
  `electron`, `@codemirror/*`, `@lezer/*`, Node builtins.
- **Embedded assets:** an esbuild plugin with an `embedded:` namespace that
  gzips and base64-encodes WASM, fonts and a separately bundled IIFE worker
  into `main.js` (O2T `build/embed.mjs`, GCC `build/build-plugin.mjs`).
- **Pin every dependency exactly** (no `^`), and justify each in
  `docs/technical/dependencies.md`. A dev-tool change can alter the shipped
  file; compare checksums (SCAR-042).
- **Licences from what is bundled:** read packages from the esbuild
  metafiles of all bundles, not from `package.json`; refuse a disallowed
  licence; generate and ship `THIRD_PARTY_NOTICES.md`. WASM binaries embed
  Rust crates whose licences npm does not carry: say so (GCC, O2T,
  SCAR-036).
- **Release-content checks:** fail if `main.js` names the build folder or
  contains test-build diagnostics (GCC `build/release-checks.mjs`).
- **ESLint:** `typescript-eslint`, `eslint-plugin-obsidianmd` (OVC),
  Prettier `--check`.

## Tests

- **Runners:** vitest (GCC, O2T); `node --test --experimental-strip-types`
  (OVC).
- **Obsidian in tests**, two working approaches:
  - structural interfaces plus a fake DOM and an in-memory store, no module
    mock (GCC `tests/support/fake-dom.ts`, `memory-store.ts`);
  - `vi.mock('obsidian', …)` with fakes for Notice, Setting, Modal and a
    FakeEl DOM, plus `fake-vault.ts` and `fake-worker.ts` (O2T
    `tests/support/fake-obsidian.ts`).
  A fake DOM has no layout: layout claims need a manual check.
- **Integration tests in Node** run the real compile (Typst WASM) on a
  generated fixture vault and read the output back (text in the PDF, not
  just "file exists": SCAR-015). Raise time limits for heavy tests
  (SCAR-030).
- **Fixture vaults are generated** (`npm run fixtures`,
  `tests/fixtures/generate*.ts`) and change only through the generator;
  never real data (SCAR-035). OVC asserts the SHA-256 of every fixture file
  is unchanged after a run.
- **Expected outputs checked in** and compared by line diff (OVC).
- **Text checks over the sources** as guards: only one module writes files;
  `obsidian` imported only in `main.ts`; manual-check index and files agree;
  the warning-code document matches the code.
- **Proving the tests:** a throwaway reference implementation shows the
  tests can pass; then mutation checks by hand on the real code, counted per
  ticket ("14 of 14 caught"); a surviving mutant gets a new test (SCAR-023).
  Back up with full relative paths and a checksum list, in `bash`
  (SCAR-018).
- **Non-default values:** every configurable rule is tested at a value
  whose result differs from the default's (SCAR-019).
- **Performance gates:** e.g. 1,000 notes plus 200 assets (OVC); fuzz tests.

## Quality gate

Typical: `npm run typecheck` (strict), `npm run build`, `npm run lint`,
`npm test`, `npm run test:integration`. All must run before a task is
declared done; a step that did not run is not reported as passed (rule 3,
SCAR-006).

## Manual checks

- **One file per check:** `docs/technical/manual-checks/Mnn.md`, index in
  `manual-checks.md` (SCAR-029).
- **Each check has:** When, Needs, **Expected version (stop if it
  differs)**, numbered steps naming the real controls, what the developer
  prepared, known limits ("not measured"), and Results with facts verified
  on disk marked separately.
- **Before handover:** decisions written down (SCAR-032), each step walked
  in the real app, a Node rehearsal, a checksum list of the vault, backup of
  `data.json` and plugin state (SCAR-012, SCAR-018).
- **After the run:** compare the disk with the expected files and sizes
  (SCAR-015), keep the outputs, undo and prove it: same file count, every
  checksum equal, no unknown file (`shasum -c`; "315 of 315 equal").
- **Spikes with candidates** state the selection rule before the run
  (GCC M03, SCAR-020).
- **Record the environment** (machine, OS, Obsidian version) in every
  result (SCAR-045).

## Scratch-vault install

`npm run install:scratch -- <vault> --version x.y.z-mNN.k`
(`build/install-scratch.mjs`, GCC and O2T): builds, checks the target has
`.obsidian`, copies only `main.js`, `manifest.json` (with the test version)
and `styles.css`, refuses a version already installed without `--force`,
never touches `data.json`, and proves by SHA-256 that the installed files
equal the fresh build. A real vault gets a plugin build only after a checked
snapshot (SCAR-018).

OVC: `npm run deploy` copies to the fixture vault or `OBSIDIAN_PLUGIN_DIR`.

## Release

1. Bump the version in `package.json`, the lock file, `manifest.json` and
   `versions.json` (OVC: `version-bump.mjs` syncs them).
2. Write `releases/vX.Y.Z.md`: date, Features, Changed behaviour, Fixes,
   Breaking changes and migration, Known limits (**Tested on / Not
   tested**), stories and issues, test counts; add the line in
   `releases/RELEASES.md`.
3. Commit, then `npm run release:build` (GCC, O2T `build/release.mjs`):
   refuses uncommitted changes, clones HEAD to a temp folder, `npm ci`, full
   gate, build, checks manifests agree and the notes are dated and indexed,
   writes `dist/release/<v>/` with `main.js`, `manifest.json`,
   `styles.css`, `LICENSE`, `THIRD_PARTY_NOTICES.md`, a deterministic zip and
   `SHA256SUMS.txt`. It **never tags, pushes or publishes**.
4. On the owner's word: tag the exact version (no `v`), push,
   `gh release create` with the files attached; close the issues naming the
   version.
5. The released `main.js` hash equals the one the manual check ran on
   (recorded in the SCAR ledger).

## Headless Obsidian (WPE)

- Docker with upstream's Dockerfile: Obsidian under xvfb, the plugin loaded
  via `electron-injector`, a script (`tools/headless/export-report.mjs`) that
  runs the exporter and writes its own run report (the injector panics after
  the script on 1.14.5, so the console log is not usable).
- The automated tests read only a generated fixture vault with a seeded
  generator, third-party plugins fetched by a lock file with checksums, and
  a tiny own fixture plugin that draws in steps.
- **Limits:** the harness reproduced the content loss (about 1 run in 2) but
  never the desktop freeze, and its Obsidian version differs from the
  user's. A green headless run is evidence for that version and machine
  only (SCAR-045, SCAR-032).
- **DOM library for tests:** jsdom parses `<div>` inside `<p>` like
  Chromium; happy-dom does not and was rejected. Count tags with a parser,
  not a regex (SCAR-028).
- Parallel integration runs gave runner RPC timeouts and incomparable
  timings: run heavy tests serially with explicit limits (SCAR-030).

## Maintaining a fork of an upstream plugin (WPE)

- Keep upstream directories close to upstream so fixes still merge; new
  code goes into new directories (`src/core/**`).
- List every change to an upstream file in a "Changes to upstream files"
  table (`docs/technical/dependencies.md`); record the upstream base commit.
  The release gate reads it, so CI clones need `fetch-depth: 0`.
- Lint upstream code against a recorded baseline count (excluding generated
  files, SCAR-043), new code strictly.
- Keep a security bump of a runtime dependency an explicit decision with its
  diff size noted (SCAR-042).

## Private repository with a public copy (OVC)

`development/scripts/prepare-github-publish.sh` rebuilds `github-publish/`
(a clone of the public repo) from `git ls-files --recurse-submodules` only,
so ignored files (`docs/state/`, `CLAUDE.local.md`, `.obsidian/`) never
leak; copies a public README and issue templates; renames personal names;
greps for forbidden words, also inside the release zips, and fails if any
remain. The owner reviews `git diff` there before pushing.

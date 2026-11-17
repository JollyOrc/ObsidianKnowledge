# Obsidian API: findings and pitfalls

Source plugins: GCC (Game Card Creator), O2T (Obsidian2Typst), OVC (Obsidian
Vault Compiler), WPE (obsidian-webpage-export fork). Measured on Obsidian 1.15.0, macOS, Apple silicon unless
stated.

## Plugin structure

- **Keep `main.ts` a thin shell.** It builds dependencies and registers
  commands, menus, views and processors; every decision lives in a function
  that takes its dependencies as arguments. No test reaches `main.ts`
  (SCAR-031). GCC: `src/plugin/main.ts`, `obsidian-deps.ts`; OVC: pure
  `InMemoryVaultCompiler` (files map plus an injected `resolve()`), `main.ts`
  only reads the vault and wires it.
- **Pure core, enforced by lint.** `src/core/**` may not import `obsidian`,
  `electron`, `node:*`, `fs` or `path` (`no-restricted-imports` in
  `eslint.config.mjs`, GCC and O2T). GCC additionally text-checks that
  `from 'obsidian'` appears only in `main.ts`.
- **Structural types instead of the Obsidian classes** (`AppLike`,
  `ModalLike`, `MenuLike`, GCC) make the plugin layer testable without
  mocking the `obsidian` module.
- **`onunload` disposes everything** that outlives a call: workers, preview
  controllers, stores, pending debounced saves (O2T, GCC, OVC).

## Vault, adapter and files

- **Read notes with `vault.read`**, not the cache, when the content matters
  (GCC). Binary assets: `vault.readBinary` (OVC).
- **File or folder?** `getAbstractFileByPath` plus a duck-type
  (`extension` / `children`), or `getFileByPath` / `getFolderByPath` (GCC,
  O2T). Folders: `vault.getAllFolders`; notes: `getMarkdownFiles`.
- **Hidden or non-indexed folders** (fonts, icons, anything under a dot
  folder) are not in the vault index: read them through `vault.adapter`
  (`exists`, `read`, `readBinary`, `write`) with `normalizePath` (O2T).
- **Rename and delete through `app.fileManager`**, so links are updated and
  the user's trash setting is respected (O2T `vault-port.ts`).
- **`createFolder` part by part** for nested folders (O2T).
- **One writer for all output.** Inside the vault: `vault.create`, `modify`,
  `createFolder`, `createBinary`; outside: Node `fs`. Everything goes through
  one module, and a text check over the sources fails on any other write call
  (OVC `output-writer.ts`, GCC `disk-writer.ts`, SCAR-033).
- **Classify a user path before writing** (SCAR-033): absolute, relative,
  `..`, `~`, the vault root, `.obsidian`, and a sibling folder that only
  shares the vault root's prefix (`startsWith(vaultRoot)` is wrong). A path
  joined as text lost its leading `/` and wrote into the vault (OVC).
- **Vault root on disk:** `adapter.getBasePath()` exists on desktop only;
  feature-detect it and fail with a clear "desktop only" message (GCC, OVC).
- **Node `fs` only on desktop:** `window.require('fs')` behind an
  `isDesktopRuntime()` check; on mobile both are null and the feature
  degrades (OVC).

## Settings and `data.json`

- **Merge loaded data over the defaults**, migrate in place, keep unknown
  keys, ignore a key of the wrong type and name it in a Notice; a
  non-object gives the defaults (GCC `settings-file.ts`, OVC).
- **Read settings when a run starts**, not once at load (GCC).
- **Do not save on every key press.** Either commit on blur or Enter (O2T
  `settings/commit.ts`, GCC) or debounce (OVC `delayed-save.ts`, 500 ms) and
  flush in the settings tab's `hide()` and in `onunload()`.
- **Back up before the first write** of a new settings screen: GCC copies
  `data.json` to a backup file once and aborts the write if the copy fails.
- **A hand edit of `data.json` while the tab is open** survived a later
  commit in GCC because only the changed key is written (M04, measured).
- **`saveData` writes two-space indented JSON** (observed once, 1.15.0, GCC).
- **The settings tab jumps to the top on redraw**; keep and restore the
  scroll position (O2T `settings/scroll.ts`).
- **Startup order:** stores that need the vault index load in
  `workspace.onLayoutReady`, and must announce when they are loaded, or a
  dropdown drawn earlier stays empty (O2T M11).

## Menus and commands

- **`file-menu` fires for the file explorer, a note's tab header menu and
  the editor's "more options"**; `files-menu` is the multi-select variant
  (GCC, measured). Every place must be listed and checked (SCAR-017).
- **Menu callbacks are synchronous.** Data that needs an `await`
  (`readBinary`, a file read) cannot decide whether an item is shown
  (SCAR-022, O2T). Build the menu from what is known; read at the click.
- **Read the path at the click**, not when the menu is built (GCC `ui.ts`).
- **Right-click does not move the editor cursor:** `editor-menu` items act on
  the cursor line, not the clicked line (O2T M12).
- **`checkCallback`:** with `checking = true` only answer, never run (GCC).
- **Prefix menu item titles with the plugin name** when several plugins add
  items to the same menu (O2T).

## Modals

- **Subclass `Modal`, override `onOpen` / `onClose`.** Closing by the X or a
  click outside must count as "cancel"; the first answer wins (GCC).
- **Enter confirms, Escape cancels, and Enter is ignored while an IME is
  composing:** `event.key === 'Enter' && !event.isComposing &&
  event.keyCode !== 229` (OVC `prompt-keys.ts`; same rule in O2T and GCC
  `dialogKey`). SCAR-013.
- **Obsidian focuses the first focusable element of a modal after `onOpen`**
  (`Modal.open()`, measured on 1.15.0, GCC M12 diagnosis): a `focus()` call in
  `onOpen` is overridden within 0 to 1 ms. So **put the safe button first in
  the DOM** (`[Cancel] [Replace]`) instead of focusing it. The focus ring only
  shows after keyboard input; after a mouse click the focus is moved just the
  same, invisibly, and Enter acts on it (GCC #58, SCAR-014). A test fake must
  copy this behaviour or it cannot see the defect (SCAR-028).
- **A one-off dialog does not write settings:** values typed for one run
  (export path, chosen note) must not leak into `data.json`, also not on
  Cancel (OVC 0.20.1).
- **Live values in dialogs:** a "resolved path" line must update as the user
  types (SCAR-016, OVC).
- **A side panel beside a modal works:** a second `.modal` element inserted
  into `containerEl` before `modalEl` (relative, z-index 1, 25em wide) shows
  beside the dialog, scrolls, takes typing, and goes away with the dialog on
  Escape, the X and a click outside (GCC M11, Obsidian 1.15.0).
- **Obsidian's modal takes Escape first:** a `keydown` listener on a field in
  that panel never saw Escape; the dialog closed anyway. Do not rely on a
  field's own listener for Escape; Enter in such a field was not measured
  (GCC M11).
- **Opening a plugin's settings tab:** `app.setting.open()` then
  `app.setting.openTabById('<plugin id>')` (private, not in the typings)
  opens the tab (GCC M11, Obsidian 1.15.0). Reach it through a structural
  type and feature-detect it.

## Markdown rendering

- **`registerMarkdownCodeBlockProcessor`** for own blocks (O2T). The
  block's position comes from `ctx.getSectionInfo(el)`, which **can return
  null**; every caller handles that.
- **Redraw after a settings change:** reading views via
  `view.previewMode.rerender(true)`; live preview via
  `app.workspace.updateOptions()` (O2T `main.ts`; the live-preview path is
  covered by a manual check, not measured in code).
- **Read the active preset/settings on every draw**, never cache them at
  registration (O2T, SCAR-016).
- **An invalid block stays visible** as raw text with a notice above it
  (O2T); never an empty element.

## Rendering a whole note to HTML (WPE)

- **The reading view draws only the sections near the visible area.** In a
  500 px window a 15-section note had 10 sections drawn; scrolling removes
  sections scrolled past. Waiting for "all sections" therefore times out.
  Growing the preview to a huge height fixed it headless but **froze the
  desktop** (SCAR-045).
- **Render off screen instead:** `MarkdownRenderer.render(app, markdown,
  hostEl, sourcePath, component)` into an absolutely positioned host
  (900 px wide, removed afterwards) draws every section and embed. The
  `Component` must be `load()`ed before and `unload()`ed after; pass the
  note's real path as `sourcePath` or relative links break
  (`src/plugin/render-api/render-api.ts`). Measured on the desktop: 331 of
  331 notes, no freeze, 40 s instead of 108 to 121 s.
- **Copy rendered DOM with `cloneNode`, not `innerHTML`:** re-parsing HTML
  moves block content out of `<p>` (a `<div>` inside a `<p>` is parsed out
  by the HTML spec), which emptied embeds.
- **Embeds fill asynchronously:** poll (50 ms) until no `.markdown-embed`
  lacks content (no `.markdown-embed-content`, or one without text, svg or
  img), at most 5 000 ms per note; what is still empty gets a placeholder
  and a warning.
- **Other plugins' post-processors and code-block renderers give no
  completion signal.** "Block not empty" is the wrong test: a plugin may
  insert a `<style>` first (a Dataview `dv.view` block held only its
  `<style>` in 117 of 117 renders). Count a `block-language-*` element as
  finished when it holds a node other than style, script, link, comment or
  whitespace **and** a `MutationObserver` (subtree, attributes, text) sees no
  change for 150 ms; overall limit 5 000 ms; at the limit one warning per
  block, `no output` or `still changing` (`src/core/render/blocks.ts`). Lost
  views went from 2 to 3 of 39 per export to 0; the waiting doubled page
  writing time (27 s to 52 s).
- **Measure each other plugin's behaviour:** a linker plugin's "link only
  once" turned out to be per post-processor call, not per note
  (`docs/technical/spike-virtual-linker.md`, SCAR-024).
- **A running Obsidian overwrites `data.json`:** edit it by hand only with
  Obsidian closed.

## Links, embeds, frontmatter (OVC)

- **Resolve with `metadataCache.getFirstLinkpathDest(link, sourcePath)`**
  after stripping `!` and `#heading`, inside try/catch; fall back to a
  deterministic basename search (smallest path wins) with a warning.
- **Percent-decoding must never throw:** a file named `50% off.md` raised
  "URI malformed" and aborted the run.
- **Markdown links to notes come in many shapes:** `%20`, relative paths,
  `<...>` brackets, missing `.md`, `#Heading%20x`, image titles
  `![a](x.png "t")`. Treat all of them like wiki links.
- **Obsidian finds headings by their text, not by a slug.** A generated link
  meant for Obsidian must use the heading text (SCAR-040).
- **Remove the frontmatter before classifying a note** by its first line
  (SCAR-034); keep `\r\n` and the original line numbers so warnings point to
  the right line.
- **Line-start rules must see past `>`:** code fences, headings and embeds
  inside quotes and callouts; fences with `~~~`, four or more backticks, or
  never closed (SCAR-041).
- `#^block` links: not implemented anywhere yet (OVC issue #100).
- Obsidian offers `parseYaml` / `stringifyYaml`; O2T uses the `yaml` package
  because it reports line and column for errors.

## Inter-plugin APIs

- **Expose:** `app.plugins.plugins['<id>'].api`, with a version field and a
  result union `{ ok: true, … } | { ok: false, reason, message }` (O2T
  `composeApi`). Read live state at every call.
- **Consume:** `app.plugins.plugins[id]?.api` is not in the typings; it
  lists **enabled** plugins only. Read it at every call, never cache it, and
  check `typeof fn === 'function'`; degrade with a coded warning when it is
  missing (GCC, OVC, SCAR-016).
- **Write the contract once, in the reader's story**; the writer's story
  points to it (SCAR-040).
- **Another plugin's defaults can override the user** if it appends them
  after the user's input (OVC with a map plugin, SCAR-037). Measure what the
  other plugin does (SCAR-024).
- **Drawing with another plugin's renderer** into a detached container, then
  rasterising via an off-screen canvas, needs a paint tick and is async
  (OVC). Base64 without Node's `Buffer` keeps it mobile-safe.
- **Fonts that exist only inside Obsidian** (e.g. Inter) are not system
  fonts: an SVG rasterised to PNG falls back to Times unless the font is
  embedded and a generic family appended (OVC 0.14.1).

## Workers, WASM, heavy assets (O2T, GCC)

- **Only `main.js`, `manifest.json` and `styles.css` are installed** from a
  release; extra files are not promised. Embed WASM, fonts and the worker
  source in `main.js` as gzip + base64 via an esbuild plugin (`embedded:`
  namespace). `main.js` grows to 15 to 30 MB; that worked.
- **Start a worker from a Blob URL**, revoke the URL once the worker has
  the code (`start-worker.ts`). Bundle the worker separately as an IIFE.
- **Blocking WASM calls belong in a worker.** O2T measured a 34 ms main-thread
  gap with the worker (limit 100 ms). Decoding the embedded payload on the
  main thread caused a ~150 ms block on the first compile in GCC; the fix
  decodes inside the worker.
- **A timer-based measurement lies while the event loop is blocked**
  (O2T spike: discarded memory sample). Measure the longest event-loop gap
  instead (O2T developer command `Check compiler`).
- **Long-lived WASM instances can degrade:** the Typst WASM panicked after
  about four whole-book compiles in one instance; O2T uses one worker per
  compile (restart ~193 ms). GCC creates the worker on first use, reuses it,
  replaces it after an error, terminates it in `onunload`.
- **Memory:** a 263-page book took Obsidian from 322 MB to 2.5 GB (4.3 GB
  with preview). Downsample preview images; free the worker after use.
- **No network:** disable default font downloads of bundled libraries
  (Typst: `loadFonts([], { assets: false })`), then verify with an empty
  network tab (GCC M02).
- **Node output = Obsidian output:** the same compile gave a byte-identical
  PDF in Node and in Obsidian (GCC M02), so Node integration tests are
  trustworthy for output bytes, not for layout or UI.
- **Canvas rendering:** pass the 2D context, size the canvas to page size
  times pixel-per-point; a guessed argument gave a blank canvas, which says
  nothing about the library (SCAR-008).

## Desktop-only features

- **System folder dialog:** `window.require('electron').remote.dialog
  .showOpenDialog`, then `@electron/remote`. Measured on Electron 43 /
  Obsidian 1.15.0 (GCC `docs/technical/manual-checks/M03.md`): both give an
  absolute path (also for an empty folder), `canceled: true` on cancel, and
  honour `defaultPath`. `<input webkitdirectory>` gives no path for an empty
  folder; `showDirectoryPicker` gives a name only and throws on cancel. How
  long `electron.remote` stays available is unknown: hide the button if
  neither exists, keep a text field that always works.
- **`isDesktopOnly: true`** in `manifest.json` when Node `fs`, Electron or
  absolute paths are needed (GCC, O2T).

## `minAppVersion`

- Set it to the highest `@since` of the APIs used, read from the pinned
  `obsidian` type definitions (GCC: 1.6.6). Say in the release notes that
  only the test environment's version was run (SCAR-032).

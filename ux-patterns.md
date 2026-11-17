# UX patterns and solutions

Patterns that held up in at least one plugin, with where to find a working
implementation. Source plugins: GCC, O2T, OVC (see the README).

## Dialogs and confirmations

- **Enter confirms, Escape cancels, IME composition ignored**, through one
  pure key function that tests can call (`dialogKey` GCC, `isConfirmKey` OVC
  `prompt-keys.ts`). SCAR-013.
- **Destructive or overwriting actions ask first** (replace output, reset,
  delete preset). The buttons carry their own names ("Replace", "Keep"),
  never "OK"; the safe button has focus. SCAR-014.
- **A reset dialog ignores Enter and Escape unless a button has focus**
  (GCC M04), so a stray key does not reset.
- **Closing a dialog any other way means Cancel**; the first answer counts
  (GCC).
- **Dialogs for one run never write settings** (OVC).
- **Show the live result of the input** (e.g. the resolved output path) as
  the user types (OVC, SCAR-016).

## Settings screens

- **Commit as you go:** on Enter or blur, not on every key. A refused value
  shows its reason in red under the field, and the old value stays saved
  (GCC, O2T).
- **Status line:** "Ready" or "Not ready: reason" at the top (GCC).
- **Sub pages with a Back button** for large settings; **"Reset this page
  to defaults"** behind a confirmation (GCC). Per-field reset icons (O2T).
- **Conditional fields** appear only while their toggle is on; the hidden
  value is kept (GCC).
- **Suggestions from the vault** (folders) in folder fields; a folder that
  does not exist is a warning, an empty or duplicate entry is refused (GCC).
- **The tab must work while it stays open:** no control may need a reopen
  to show the current state (SCAR-016). Keep the scroll position on redraw
  (O2T).
- **Folder picker:** a text field that always works, plus a
  `Choose folder…` button only where a system dialog is available, starting
  at the stored folder (GCC; see [obsidian-api.md](obsidian-api.md#desktop-only-features)).
- **Presets as notes:** user-editable presets kept as Markdown notes with
  YAML frontmatter in a vault folder; errors with line and column (O2T).

## Warnings and errors

- **Problems are warnings with a source location, never a silent abort**
  (shared rule 8). One failing note does not stop the rest.
- **Stable warning codes** (`SKILL_GLYPH_MISSING`, `STAT_SUM_MISMATCH`, …)
  documented in one file; a test compares that file with the codes the code
  can produce (O2T `docs/technical/warning-codes.md`).
- **A warnings file next to the output** (`<name> warnings.md`), plus a
  Notice with the count (GCC, O2T, OVC). OVC also writes warnings as HTML
  comments at the place in the output, and names note and source line
  (counted including frontmatter).
- **One warning per note per kind of problem**, not one per occurrence
  (GCC).
- **Failure becomes a visible placeholder:** a failed render keeps the
  original block plus a warning; an unclosed fence is closed at the end of
  the note with a warning naming its line (OVC). An invalid code block stays
  visible with a notice above it (O2T).
- **Every wait has a time limit** and reports instead of hanging: "Plugin
  block <class> in <note> did not finish within N ms (<reason>)" (WPE).
- **Reported text goes in via `textContent`**, so `<` and `&` show literally
  (WPE; shared rule 6).
- **Keep a behaviour users rely on as a setting** when replacing it (WPE
  kept its fallback renderer as an option, SCAR-027).
- **Graceful degradation:** a missing or disabled dependency plugin gives a
  coded warning and a reduced result, not an error (GCC).

## Notices

- **A refused run says what and where to fix it:** "… See Settings → <page>
  (file: <path>)" (GCC).
- **A settings-file problem names the ignored key** (GCC).
- Notices are for short messages; details go to the warnings file.

## Output

- **Never modify the user's notes.** Output goes to a configured location;
  replace only after asking (shared rule 2).
- **Assets next to the output**, deduplicated by content hash, renamed
  `_1`, `_2` on clash (OVC; SCAR-011 for generated names).
- **Deterministic output:** fixed document date, so two runs give
  byte-identical files (GCC, rule 7).
- **Credit lines** for bundled fonts and icons printed where the licence
  needs it (GCC).

## Previews and panels

- **Preview draws the same model as the export**, so the two cannot drift
  (O2T, SCAR-017).
- **Side panel view** (`registerView`) with a debounced preview and request
  ids; results of stale requests are dropped; a pause/refresh control (O2T).
- **Inspect rendered output at 100 % and 400 %** before accepting it
  (SCAR-025).

## Menus

- Offer the action wherever the user will look: explorer, tab header,
  editor "more options" (all fire `file-menu`), multi-select via
  `files-menu` (GCC).
- A command with `checkCallback` for keyboard users (GCC).
- Rebuild a ribbon menu on each click so it shows the state of now (O2T).

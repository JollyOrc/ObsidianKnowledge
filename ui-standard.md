# Preferred UI standard

The UI norm for our Obsidian plugins. A new plugin, or a new or changed
screen, dialog, menu or notice in an existing one, follows this file.

**Where it comes from.** The layout (entry points, run dialog, selection
tree, settings tab, progress tab) is taken from obsidian-webpage-export
(UI from upstream 1.9.2, unchanged in our fork 1.10.1). The behaviour rules
come from our SCARs and Obsidian's plugin guidelines; where the reference
does it differently, this file is the norm and the reference is the
exception (section 8). Other patterns that held up are in
[ux-patterns.md](ux-patterns.md); where both cover the same thing, this file
decides.

Reference files for the layout (obsidian-webpage-export repository):
`src/plugin/main.ts`, `src/plugin/settings/export-modal.ts`,
`src/plugin/features/file-picker.ts`, `src/plugin/settings/settings.ts`,
`src/plugin/settings/settings-components.ts`,
`src/plugin/render-api/render-api.ts`, `styles.css`.

## 0. Rules for every screen

1. **Keyboard.** Every dialog confirms with Enter and cancels with Escape;
   Enter is ignored while an IME composes (`!event.isComposing &&
   event.keyCode !== 229`). One pure key function, tested (SCAR-013).
2. **Destructive actions ask first.** The confirmation names its buttons
   ("Cancel", "Clear cache"; never "OK"), and the safe button has focus
   (SCAR-014).
3. **State of now.** Every control shows and acts on the current state;
   nothing needs a reopen to be right. Handlers read state when they run,
   not when they were drawn (SCAR-016).
4. **Text commits on Enter or blur**, never on every key press. Toggles and
   dropdowns save at once (SCAR-017).
5. **Refused input stays visible:** a red line under the field says why,
   and the last valid value stays saved.
6. **Styles in `styles.css`** under plugin-prefixed classes, with
   Obsidian's CSS variables (`--color-red`, `--interactive-accent`,
   `--divider-color`, `--background-secondary-alt`, `--font-ui-smaller`),
   so every theme works. No inline styles.
7. **No network** from the UI: no remote images, fonts or scripts.
8. **One string table** for every user-facing text.
9. **User text via `textContent`**, never `innerHTML` (shared rule 6).
10. **Paths compared by segments**, never by string prefix; a user path is
    classified before anything is written (SCAR-033).
11. **Every place is listed and checked:** a pattern used in several
    screens or menus is fixed in all of them (SCAR-017).

## 1. Entry points

Offer the main action in every place a user looks for it:

| Entry | Behaviour |
|---|---|
| Ribbon icon | Opens the run dialog; **or** (allowed variant, when users already rely on it) opens a menu whose **first item "Export…" opens the run dialog** |
| Command "… using previous settings" | Runs at once with the saved selection and destination, no dialog; if something required is missing, a Notice says what to set |
| Command "… current file using previous settings" | Runs for the active note; a Notice if none is open |
| Command "Open … dialog" | Opens the run dialog |
| `file-menu` on a file or a folder; `files-menu` on a multi-selection | Opens the run dialog preselected with those notes; the dialog title names them ("Export *name*") |

- A folder's notes are the files whose **path segments** start with the
  folder's segments (`A/` does not include `A 2/`).
- `file-menu` fires in the file explorer, a note's tab header menu and the
  editor's "more options"; check every one
  ([obsidian-api.md](obsidian-api.md#menus-and-commands)).
- Menu callbacks are synchronous: build items from what is known, read at
  the click (SCAR-022).
- Keep a saved selection valid when notes move: update stored paths on
  `vault.on("rename")`.
- Bringing an existing plugin in line never removes an entry users rely on
  without saying so; it stays as a "using previous settings" entry or as
  an allowed variant (SCAR-027).
- **Progress tab or Notice:** a run that takes more than about 2 s on the
  test machine (measured as a whole run) gets the progress tab (section
  5); a shorter one reports by Notice.

## 2. The run dialog

One modal holds everything one run needs, top to bottom:

1. **Title** naming what will run.
2. **Update banner** (only if a newer version is known): current → latest
   in `--color-red` on `--interactive-normal`, release note below in
   `--font-ui-smaller`.
3. **Preset dropdown as a heading row** (`Setting.setHeading()`), with the
   chosen preset's description under it. Changing the preset applies it to
   the dialog at once.
4. **Maintenance actions** with their description, each behind a
   confirmation (rule 0.2).
5. **Option toggles** for this run.
6. **Destination row:** full-width path field with placeholder, **Browse**
   (system folder dialog where available; the field always works without
   it), and the **main action button** at the end of the row. The action
   button is disabled while the destination is invalid; the red line under
   the field says why.
7. **What is at the destination**, live: nothing yet, output of an older
   version, or name, file count and date of the existing output. Updated on
   every change of the path.
8. **"More options"** row with a gear that opens the plugin's settings tab
   (`app.setting.open()`, `app.setting.openTabById(<id>)`).

Beside the dialog, a **selection panel** (a second `.modal` element before
the dialog, 25em wide, scrolling) holds the selection tree (section 3) and a
**Save selection** button.

Behaviour:

- Enter runs the action when it is enabled; Escape, the X and a click
  outside cancel. The first answer counts.
- Values chosen in the dialog are saved **when the action runs**, and the
  selection also by **Save selection**. Cancel saves nothing.
- The caller awaits a promise that `onClose` resolves with one result
  object, e.g. `{ canceled, pickedFiles, destination }`; no polling.
- An overwrite of existing output asks first (rule 0.2).

## 3. The selection tree

A file tree with checkboxes:

- A **Select all** row in the tree header; saved as "all", so notes added
  later are included next time.
- Folders start **collapsed**; checking a folder checks its contents.
- **File type tags** on non-Markdown files (no tag for `.md`).
- **Block and allow lists** hide files that must not be chosen (the
  plugin's own files, configured special files).
- Opens with the files the run was started with, else the saved selection.

## 4. The settings tab

Top to bottom:

1. **Header row** (no `<h2>` with the plugin name): help links on the
   left, a **Copy debug info** button (bug icon) on the right that copies
   plugin version, Obsidian version, settings, enabled plugins and the
   recent log, then shows a Notice.
2. **Sections** as collapsible `<details>` with a `chevron-right` icon and
   a heading `Setting` (name and description) as the summary, separated by
   a divider (`--interactive-accent`, 0.5 opacity).
3. **Feature rows:** name, description, toggle, and a **gear** that opens a
   modal with that feature's own settings. A feature that cannot be used
   now is **disabled with a tooltip saying why**. A row without
   sub-settings keeps an invisible gear so the toggles line up.
4. **Declared settings:** each option declares its name, description,
   visibility and kind (toggle, text, number, dropdown, path, list); the
   tab is generated from the declarations. Lists get **+ / −** buttons and
   show their length.
5. **Field kinds:** toggle, text with validation, dropdown, path (text plus
   **Browse**, platform-safe, optionally relative to the vault).
6. **Reset** per section to defaults, behind a confirmation.
7. **Diagnostics last** (log level and similar).

Behaviour:

- Rules 0.3 to 0.5: commit on Enter or blur; refused values explained in
  red, the old value kept; a change that affects another row updates it
  at once; the scroll position survives a redraw.
- `data.json` is read merged over the defaults; unknown keys are kept; a
  key of the wrong type is ignored and named in a Notice.

## 5. Progress for runs the user waits for

- A **progress tab** opens beside the user's tabs: heading with the
  current step, a `<progress>` bar, a sub-message with the current item,
  and a **Cancel** button. The bar is mirrored on the OS taskbar or dock
  (`setProgressBar`).
- A **log column** appears on the first warning: one entry per event,
  bold title, message below, a 5 px left border in the event's colour
  (yellow skip, red error); new entries scroll into view.
- **Cancel** stops after the current item, shows a "cancelled" Notice, and
  turns into a **Close** button; the log stays readable.
- Every warning also goes to the **warnings file** next to the output, with
  its source location; one failing item does not stop the rest (shared
  rule 8, [ux-patterns.md](ux-patterns.md#warnings-and-errors)).
- The plugin does not resize or move the user's windows.

## 6. Notices

| Situation | Text | Time |
|---|---|---|
| Finished | "✅ Finished *action*:" and the output path; the warning count if any | 5 s |
| Cancelled | "❌ *Action* cancelled" | 5 s |
| Failed | "❌ *Action* failed: *reason*" (also in the log) | 5 s |
| Missing precondition | What to set and where ("… See Settings → *section*") | 5 s |

## 7. Checklist for a story that adds or changes UI

- [ ] Which entry points (section 1), and every `file-menu` place checked
- [ ] Enter, Escape, IME, focus, confirmation for destructive actions
- [ ] Commit on Enter or blur; refused value explained; old value kept
- [ ] Every control right while the screen stays open (a test or manual
      step that changes state with the screen open)
- [ ] Styles in `styles.css`, Obsidian variables, no network
- [ ] Strings in the string table; user text via `textContent`
- [ ] Paths by segments; user paths classified before a write
- [ ] Warnings with source location in the warnings file
- [ ] A manual check in Obsidian for what a fake DOM cannot show (layout,
      focus, scrolling)

## 8. Where the reference plugin differs

obsidian-webpage-export is the source of the layout but does not meet this
norm in these points (state of 1.10.1):

| Reference behaviour | Norm | Rule |
|---|---|---|
| Folder files picked by `startsWith(folder.path)` | Path segments | 0.10 |
| Dialogs without Enter/Escape handling; no focused safe button | Enter, Escape, IME guard, safe focus | 0.1, 0.2 |
| Dialog path saved on every key press, also on Cancel | Saved when the action runs | 2 |
| Text settings saved on every key press | Commit on Enter or blur | 0.4 |
| Modal close awaited by polling a flag | Promise resolved in `onClose` | 2 |
| Inline styles | Classes in `styles.css` | 0.6 |
| `<h2>` heading and a remote image in the settings tab | No top heading, no network | 0.7, 4 |
| Some hard-coded strings | One string table | 0.8 |
| Selection button labelled "Save" | "Save selection" | 2 |
| Debug info without versions | Plugin and Obsidian versions included | 4 |
| Log in the progress tab only | Also the warnings file | 5 |
| Log panel resizes the window | Windows untouched | 5 |

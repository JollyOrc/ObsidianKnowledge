# Problems that occurred

Each entry: what happened, the root cause, the fix, and the SCAR. Ordered by
theme, not by date. Source plugins: GCC, O2T, OVC (see the README).

## Data safety

| Problem | Root cause | Fix | SCAR |
|---|---|---|---|
| Output written into the source vault (OVC) | Picker path joined as text lost its leading `/`; `startsWith(vaultRoot)` also matched a sibling folder | Path classification module, one writer, table of example paths in the story, text check that bans other write calls | 033 |
| Real vault content in the repo and tests (OVC) | Real notes used as fixtures | Generated synthetic fixture vault; publish script greps for forbidden words, also inside release zips | 035 |
| Delete ran on one click; dialogs ignored Enter (O2T, OVC) | Dialogs specified for the mouse only | Confirmation modal, Enter/Escape handling, IME guard | 013, 014 |
| Backup before a manual check missed `data.json` (O2T M30) | Backup covered what the installer changes, not what the plugin rewrites during the run | Back up `data.json` and plugin state too; recovered by checksum | 018 |
| Mutation-run backups collided (O2T) | Two files with the same base name backed up flat | Back up with full relative paths and checksums | 018 |
| A zsh backup loop copied nothing (GCC) | `for f in $VAR` does not word-split in zsh | Run such scripts in a named shell (`bash`); verify the checksum list before changing anything | 018 |
| A one-off export dialog changed the saved settings, also on Cancel (OVC 0.20.1) | Dialog values written to the settings object | Values used for one run only | 016 |

## Output silently wrong

| Problem | Root cause | Fix | SCAR |
|---|---|---|---|
| PDF had images but no text (O2T) | No font shipped; checks tested success, not content | Bundle fonts; checks read the text back | 015 |
| Frontmatter became book content (OVC) | Note classified by its first line before the YAML was removed | Classify after removing the header | 034 |
| Heading links went nowhere (OVC) | Links written as slugs for an unnamed reader; Obsidian matches heading text; tests checked only self-consistency | Name the reading program in the story; use its form | 040 |
| Code, headings, embeds inside quotes and callouts broken (OVC) | Line-start rules did not see past `>` | Container-aware rules | 041 |
| Invisible HTML-comment markers broke a later comparison (OVC) | In-band markers read by steps they were not meant for | Markers kept out of the compared text | 039 |
| Warning count inflated (OVC) | The link lookup also reported; every caller that only looked added warnings | Split lookup and report | 038 |
| Warning line numbers off after embeds, frontmatter, generated parts (OVC 0.18 to 0.19) | Line counted on transformed text | Count on the source, including frontmatter | — |
| User's map settings silently did nothing (OVC) | The other plugin appended its defaults after the user's lines | Measure the dependency; order input accordingly | 037, 024 |
| Template rules silently did nothing (O2T) | Story claimed a Typst `import` applies rules; it brings definitions only; stood in a Done story for 5 days | Measure dependency behaviour before writing it into an AC | 024 |
| Percent sign in a file name aborted the run (OVC 0.19) | `decodeURIComponent` threw | Decoding that never throws | — |
| Every card failed when largest = smallest text size (GCC 0.2.0) | Size list written as a bare number | Always a list; test at non-default values | 019 |
| A 200-card run failed in the compile library (GCC) | Per-card call carried the whole data: quadratic growth | Shared state computed once | — |
| Visible seams between template strips (O2T M13) | Opaque gradient strips overlapped | Strips fade to transparent | 025 |

## Stale UI

| Problem | Root cause | Fix | SCAR |
|---|---|---|---|
| Font list and glyph notice appeared only after reopening the tab (O2T M5) | Controls showed the state at draw time | Read state when acting / redraw on change | 016 |
| Reset icon used a value read at draw time (O2T) | Same | Same | 016 |
| Resolved-path line showed the path from when the dialog opened (OVC) | Same | Update as the user types | 016 |
| Refused folder in the dialog failed silently (GCC M04) | Message shown, then the page redrawn at once and erased it | Redraw only when the value was saved; re-checked in Obsidian | 016, 045 |
| Preset dropdown empty after restart (O2T M11) | Store loaded in `onLayoutReady` but did not announce it | Load event; dropdown redraws | 016 |
| A notice per key press while typing a number (O2T) | Text field saved on every key | Commit on blur or Enter, everywhere it occurs | 017 |

## Process and verification

| Problem | Root cause | Fix | SCAR |
|---|---|---|---|
| Manual test run on an old build (OVC) | Same version number for different builds | Test-build versions `x.y.z-mNN.k`, checksum equal to the fresh build, "Expected version: stop if it differs" | 012 |
| A manual-check step named a control that does not exist (O2T M30) | Step not walked against the real app before handover | Walk every step in the real app; prepare what it needs | 012 |
| Manual check reported "passes" but two steps had not run as written (GCC M09) | Result not compared with the disk | Compare output files with the step list and expected sizes | 015 |
| A Node rehearsal missed warnings the real run showed (O2T M30) | Rehearsal fixture lacked the rule the real data used | Rehearse with inputs that exercise the rule | 023 |
| Fix passed headless (3 of 3 runs, full gate green) and froze on the user's desktop in three manual checks (WPE) | The approach was measured only in the Docker/xvfb harness, which never reproduced the freeze | Measure the chosen approach on the desktop at G1, before tickets; make the harness reproduce the failure first; cheap diagnosis builds (own version suffix, step log, heartbeat) | 045, 024 |
| First release-pipeline run needed three tries (WPE) | Shallow clone, older Node without type stripping, read-only token | Run a pipeline once before it is needed; compare its environment with the local one | 045, 006 |
| A dev dependency changed the shipped `main.js` with no source change (WPE) | The test runner lifted runtime packages | Record the bundle checksum before and after any dependency change; pin runtime packages | 042 |
| Lint baseline count wrong, 621 instead of 478 (WPE) | `eslint src` included a git-ignored generated bundle | Say what a count is over; exclude generated files by path | 043 |
| A whole-tree typecheck test assigned to the first ticket could not pass until the third (WPE) | Red tests of later tickets import modules that do not exist yet | At G2, assign whole-tree checks to the last ticket whose files they read | 044 |
| Export runs void: settings had reset, though the snapshot checksum matched (WPE) | The snapshot of an empty `data.json` equalled an empty source | Check a snapshot's content, not only equality; read the settings a step depends on before each run | 018, 012 |
| A stand-in fixture plugin passed under the broken rule (WPE) | It awaited in its processor, unlike the real plugin | Copy the real dependency's timing; show the test red under the old rule first | 023, 028 |
| A test reader reported labels missing that were in the HTML (WPE) | The reader replaced `<svg>` by a placeholder; a regex `<p` matched `<path` | Check the reader itself; use an HTML parser for tag counts | 028 |
| Embeds dropped in the export (WPE) | Reading view draws only visible sections; an upstream "div inside p" step emptied the first of two embeds; `innerHTML` re-parsing | Off-screen `MarkdownRenderer.render`, `cloneNode`, placeholder plus warning | 024, 045 |
| Test input could not tell the rule from its rival (GCC) | Non-default example gave the default's result | Example must differ from the default; mutation checks | 019, 023 |
| Old release zips kept a licence conflict after the dependency was swapped (OVC) | Fix applied to the build, not to shipped artifacts | Audit and replace shipped artifacts | 036 |
| A `git push` bundled into a compound command after a "go" that did not name it (GCC) | Directive scope stretched | Git only on the user's word, one action per command | rule 14 |
| A premise written as measured when only part of it was (GCC) | Claim wider than the measurement | State exactly what was measured | 032 |

# Which SCARs for which plugin work

A reading list by task. The SCARs themselves:
<https://github.com/JollyOrc/scars> (also `scars/` in each plugin). Read the
whole index at G0 anyway (shared rule step 0); this map tells you where to
look hardest.

| You are working on … | Read first | Obsidian-specific example |
|---|---|---|
| A modal or prompt | 013, 014, 016 | IME guard on Enter; safe button focused; live resolved path |
| A settings tab | 016, 019, 020, 017 | Commit on blur/Enter; no reopen needed; non-default values tested |
| A context menu or command | 022, 017, 016 | Menu callbacks are synchronous; `file-menu` fires in three places |
| Writing files | 033, 014, 018, 043 | One writer; classify the user's path; sibling-prefix folders |
| Reading notes, links, frontmatter | 034, 041, 040, 009, 026 | Strip frontmatter first; see past `>`; heading text, not slugs |
| A code-block processor or preview | 016, 017, 021, 025 | `getSectionInfo` may be null; preview and export share one model |
| Rendering notes through Obsidian, waiting on other plugins | 024, 045, 023, 028 | Off-screen `MarkdownRenderer.render`; quiet period plus time limit |
| Maintaining a fork | 001, 042, 043 | Changes to upstream files listed; baseline over source only |
| Using another plugin's API | 024, 037, 040, 022 | Read the API at every call; enabled plugins only; measure its defaults |
| Bundling WASM, fonts, workers | 008, 024, 036, 042, 045 | Blob-URL worker; no CDN downloads; licences from the metafile |
| Generated output (books, PDFs, cards) | 010, 011, 015, 021, 039 | Check content, not existence; joins between pieces; stable layout |
| Manual checks in Obsidian | 012, 015, 018, 029, 032, 045 | Test-build version; checksum list; undo proven; environment recorded |
| Tests and test tooling | 023, 028, 030, 019, 044 | Inputs that tell the rule from its rival; time limits for heavy work |
| Shell entry point / architecture | 031, 001, 038 | Decisions out of `main.ts`; lookups without side effects |
| Releases | 006, 003, 036, 042 | Release build from a clean clone; tested-on / not-tested in notes |
| Test data | 035 | Generated fixture vaults only |
| Decisions and options for the user | 020, 024, 027, 032 | Measure each option in the real Obsidian before offering it |

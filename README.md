# Obsidian Knowledge

Common knowledge for building Obsidian plugins: what the Obsidian API really
does (as measured, not as assumed), the problems we ran into and why, and the
UX, build, test and release solutions that held up. Harvested on 2026-11-17
from four plugins:

| Short | Plugin | Version read | Desktop only | minAppVersion |
|---|---|---|---|---|
| GCC | [Game Card Creator](https://github.com/JollyOrc/GameCardCreator) | 0.6.0 | yes | 1.6.6 |
| O2T | Obsidian2Typst | 0.15.2 | yes | 1.5.0 |
| OVC | Obsidian Vault Compiler | 0.24.0 | no | 1.0.0 |
| WPE | obsidian-webpage-export, our fork (upstream base 1.9.2) | 1.10.1 | yes | 1.6.0 |

Each entry names its source plugin and, where useful, the file in that
plugin's repository, so a claim can be traced back to its evidence. A claim
holds for the environment it was measured in (mostly: M1 MacBook, macOS
27.0.1, Obsidian 1.15.0); nothing here is claimed wider than that
(SCAR-032, SCAR-045).

## Contents

| File | What it holds |
|---|---|
| [obsidian-api.md](obsidian-api.md) | Obsidian API findings and pitfalls: vault and adapter, data.json, menus, modals, code blocks, links, workers and WASM, inter-plugin APIs, desktop-only features |
| [problems.md](problems.md) | Problems that occurred, with root cause, fix and the SCAR they led to |
| [ui-standard.md](ui-standard.md) | **The preferred UI standard** (the norm for every screen): rules for every screen, entry points, run dialog, selection tree, settings tab, progress tab, notices, a checklist for UI stories; layout after obsidian-webpage-export |
| [ux-patterns.md](ux-patterns.md) | UX solutions: dialogs, confirmations, settings screens, warnings, notices, folder pickers, previews |
| [tooling.md](tooling.md) | Architecture, build and bundling, tests, manual checks, scratch vaults, release, licences, public copies |
| [workflow.md](workflow.md) | The development process we follow and where its rules live |
| [scar-map.md](scar-map.md) | Which SCARs to read for which kind of plugin work |

## The SCARs and the shared rules

This repository does not copy them; it points to them.

- **SCARs** (post-incident lessons, project-independent):
  <https://github.com/JollyOrc/scars>, index in its `README.md`. Every plugin
  carries it as the submodule `scars/`.
- **Shared agent rules** (process steps 0 to 4, SCAR gates G0 to G5, agent
  rules 1 to 15): `scars/AGENT-RULES.md`.
- **Gate table** (which SCARs are re-checked at which gate): `scars/GATES.md`.

When a lesson here is a general failure mode, it belongs in a SCAR, and the
entry here links that SCAR. When it is a fact about Obsidian (an API
behaviour, a measurement), it belongs here.

## Rules for this repository

- **No real data** (SCAR-035): no notes, names, values or vault paths from a
  real vault; measurements are kept as numbers.
- **Measured, with its environment** (SCAR-024, SCAR-032): a statement about
  Obsidian says how and where it was measured, or says "not measured".
- **One fact, one place**: a fact lives in one file; others link it.
- **Adding a lesson**: name the plugin, version and file it comes from, and
  the SCAR it relates to, if any. A new general failure mode goes to the
  scars repository first (its own rules), then gets linked here.

## Licence

Released into the public domain under [The Unlicense](LICENSE).

# How we work

This file summarises where the process lives; it does not restate it. The
binding text is in the scars repository: <https://github.com/JollyOrc/scars>.

## Where the rules are

| What | Where |
|---|---|
| Process steps 0 to 4, SCAR gates G0 to G5, agent rules 1 to 15, code style, versioning | `scars/AGENT-RULES.md` |
| Which SCARs are re-checked at which gate | `scars/GATES.md` |
| The SCARs and their index | `scars/README.md`, `scars/SCAR-nnn-*.md` |
| A plugin's own context, gate commands, architecture, release steps | that plugin's `CLAUDE.md` |
| Local-only notes (paths, the real vault) | that plugin's git-ignored `CLAUDE.local.md` |

Each plugin imports the shared rules with `@scars/AGENT-RULES.md` in its
`CLAUDE.md` and carries the scars repository as the submodule `scars/`.

## The process in one screen

0. **SCAR review** of the request against every SCAR's Prevention list
   (gate G0); gaps go to the user before anything is written.
1. **User story** with acceptance criteria and concrete examples (SCAR-007),
   including the non-happy paths (SCAR-002) and non-default settings
   (SCAR-019).
2. **Tests** against the ACs; proven against a throwaway reference.
3. **Tickets** as GitHub issues, each with a SCAR-check section.
4. **Code**, then G4: full quality gate, mutation checks, manual check in
   the real Obsidian where the ACs need it.
5. **G5:** does anything found generalise into a SCAR? Then edit the scars
   repository, push on the owner's go, and bump the `scars/` pointer in
   every project.

One step per "go"; each step is reported with its evidence.

## State that survives a context reset

`docs/state/` in each plugin (git-ignored): `00_governance.md`,
`10_ticket.md`, `20_plan_approved.md`, `30_scar_checks.md` (append-only SCAR
ledger), `99_next_action.md`. Read in lexical order at the start of every
session. Cross-plugin work keeps its state in the control project.

## Working agreements worth knowing before you start

- A question is not a directive (rule 12); committed speech: assert the
  facts, promise with conditions of satisfaction, or counter-offer (rule 13).
- No commit, tag, push or release without the owner's word (rule 14).
- No real data anywhere in a repository (rule 2, SCAR-035).
- The real vault is the original, not a test vault: a plugin build goes
  there only after a checked snapshot with proven undo (SCAR-018).
- Contracts between plugins are written in the reader's story (SCAR-040).
- A fix seen in one plugin is searched for in every plugin (SCAR-017).

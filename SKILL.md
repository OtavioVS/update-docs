---
name: update-docs
description: >-
  Maintain a project's architecture documentation using a two-tier system (one
  "map" file + deep-dive docs). Use when a feature or fix has just been
  implemented and needs folding into the docs, when architecture docs are found
  stale or contradicting the code, when the user asks to update, organize, or
  reorganize documentation, when a UI/UX decision or visual change (screen,
  style, flow, copy, divergence from a design prototype) needs documenting, or
  to bootstrap architecture docs in a project that has none.
argument-hint: "[what changed, or 'bootstrap' | 'reorganize' | 'audit']"
---

# Maintain architecture documentation

You maintain a **two-tier** documentation system:

- **The map** — ONE architecture file. Holds the big picture, non-negotiable rules,
  lifecycles, layers, decisions (the *why*), and constraints. **Current state only**, with a
  pointer to a deep dive wherever more detail exists.
- **Deep dives** — files in the project's docs folder. Implementation walkthroughs,
  debugging narratives, device logs, dead ends worth not repeating, and plans. No size limit.

Everything below is a **default**. If the project has a documentation contract (Step 1.3),
its rules OVERRIDE these defaults.

## Step 1 — Locate the project's documentation

1. Check `CLAUDE.md` for a stated architecture-doc location. If found, use it.
2. Otherwise Glob for common map locations, in order:
   `agent/architecture.md`, `ARCHITECTURE.md`, `docs/ARCHITECTURE.md`,
   `docs/architecture.md`, `doc/architecture.md`.
3. Read the map's header (first ~15 lines). If it links to a rules/contract file (e.g.
   `documentingSkill.md`, `documentation-rules.md`), **read that file completely** — its
   project-specific rules override this skill's defaults.
4. **No contract found → mandatory setup, never silent defaults.** If the header links no
   contract and a Glob next to the map (`documentation-rules.md`, `documenting*.md`) finds
   none, STOP and run the user through the documentation options before any editing:
   contract file name & location, deep-dive folder & naming, budgets, mirror policy,
   UI/UX mode & token set — the fields of the contract template in
   `references/bootstrap.md` §4. Create the contract from the answers and **link it from
   the map header** (plus a pointer in `CLAUDE.md` if one exists) — if the user picks a
   non-default name, that recorded link is the ONLY way future runs will find it. This
   gate also fires when the docs claim a structure that is missing on disk (e.g. the
   declared deep-dive folder or a linked doc file doesn't exist): surface the gap and
   settle it with the user before proceeding.
5. Note whether the map has sibling **translation mirrors** (e.g. `architecturePT-BR.md`).
   If the contract declares a mirror policy, follow it; if a mirror exists with no policy,
   ask the user whether to keep syncing it before touching it.
6. Note whether the project has **UI/UX docs** (default home: `docs/uiux/`) and a
   **design-token set** — a code-level token module (e.g. `src/theme/tokens.ts`, CSS
   custom properties, a Tailwind theme) mirrored by `docs/uiux/design-tokens.md`.
   Checklist D depends on this; the contract file records the project's decision.
7. **No map found at all** → this is a bootstrap. Read `references/bootstrap.md` and follow
   it instead of the checklists below.

## Step 2 — Classify the task

| Situation | Do |
|---|---|
| A feature/fix landed and docs must reflect it | Checklist A |
| Docs found stale, wrong, or contradicting the code | Checklist B |
| Map has grown too large / user asks to reorganize | Checklist C |
| The change touches a user-facing surface (screen, component, style, flow, copy) | Checklist A **+ D** |
| A pure UI/UX decision was made (no architecture impact) | Checklist D only |
| No docs exist | `references/bootstrap.md` |

## Checklist A — a feature or fix landed

Work through every item; say explicitly which items were no-ops.

1. **Find ALL affected locations, not just one.** A change usually touches several places:
   a lifecycle step, a trigger table row, a one-line summary, the file tree. Before editing
   anything, Grep the map for every keyword the change involves (function names, events,
   the old behavior's wording — e.g. a change to logout: grep `logout`, `onLogout`,
   `un-REGISTER`) and list every hit as a candidate edit. Editing only the first match and
   stopping is the most common failure mode. Then place NEW content by kind — read the
   map's `##` headings (that list IS the skeleton):
   - new runtime behavior / state / flow → the lifecycle section(s)
   - new module, service, or interface → the layers/services section (+ its table)
   - new "why X over Y" choice → the decisions section
   - new dependency → the libraries/dependencies section
   - new build/config gotcha → the constraints section
   - new files → the file-structure section
2. **Merge, never append.** Integrate into the existing section. Do NOT add a new
   top-level `##` section — that is exceptional and needs the user's approval.
3. **Write current state only.** No "SOLVED", no version tags, no "previously X, now Y".
   Rewrite the affected paragraph to describe how the system works *now*. Move the
   discovery story (old behavior, why it failed, logs) to the feature's deep dive.
   Exception: a feature that is *built but disabled* IS current state — document the flag
   and why it is off.
4. **Deep dive.** If the feature has meaningful depth (mechanism, debugging story, caveats),
   create or update its deep-dive file. Required header:

   ```
   # <Title> — deep dive | guide | plan
   **Status:** CURRENT | PLAN | HISTORICAL
   **Summary lives in** <map path> §<n>
   <one-paragraph TL;DR>
   ```

   If a PLAN doc just shipped, flip its Status to CURRENT and fold its outcome into the map.
5. **Promote to a top rule only if it qualifies.** If the map has a rules/invariants
   section, add an entry only when breaking the invariant fails *silently* or
   *catastrophically* AND it is non-obvious. Keep that list under ~10 entries; build
   mechanics go to the constraints section instead.
6. **Tables for enumerables.** If you wrote a paragraph containing three or more parallel
   items ("on X… on Y… on Z…"), convert it to a table, one row each.
7. **One canonical home.** Before explaining a concept, Grep the map for its keywords. If
   it is already explained somewhere, link to that section instead of re-explaining.
8. **Mirror sync.** If the project has a translation mirror, retranslate the affected
   sections NOW, in this same task. Keep section numbering 1:1. Do not translate: code,
   file paths, log lines, literal identifiers, established technical terms of art. UI
   strings already in the target language stay verbatim.
9. **UI touched?** If the change altered any user-facing surface — a screen, a component,
   a style value, a flow, a UI string — run Checklist D too, in this same task.
10. **Verify** (Step 3).

## Checklist B — stale or wrong docs

1. **The code is the truth.** Where docs and code disagree, fix the docs to match the
   code — unless the doc records an *intended* behavior the code fails to deliver; in that
   case report the discrepancy to the user instead of silently rewriting either.
2. Apply Checklist A items 3–8 to every paragraph you touch.
3. Never delete a fact outright. Facts move (map → deep dive) or get corrected; if a fact
   appears genuinely obsolete, list it in your final report as removed and why.

## Checklist C — reorganize an oversized map

Budgets (defaults): map ≤ ~650 lines; any single map section ≤ ~40 lines of body text.
Budgets are **soft targets, not hard caps** — within ~5% over counts as met; stop there.

1. **Cut coarse, not fine.** Close a budget gap with ONE or TWO whole-block extractions,
   never many small edits. Extract an entire subsection, table, Mermaid diagram, or
   multi-paragraph mechanism to a deep dive in a single cut, leaving a ≤15-line summary +
   pointer. Moving individual sentences, trimming phrases, or reflowing paragraphs to
   shave lines is the known failure mode: it burns effort and barely moves the count.
   Procedure: list the map's largest extractable blocks by line count, take the biggest
   one that isn't rules/invariants, and check the count — one cut usually suffices.
2. **What stays in the map when extracting:** the invariant/behavior itself, the one-line
   why, anything that changes what a developer *does* (flags, gotchas, log tags to grep),
   and the pointer. **What moves:** mechanisms, discovery narrative, logs, dead ends,
   command playbooks — and whole diagrams or tables when the map only needs their
   conclusion, not their contents.
3. **Cheap compression** — apply these where you're already editing; never hunt lines
   with them:
   - A list or paragraph that restates an adjacent table/diagram → delete the restatement,
     keep the table.
   - 3+ parallel items in prose → table (item A6); usually shorter too.
   - Runs of blank lines or decorative separators → single blank line.
   - Verbose cross-references → `→ docs/<file>.md §<n>`.
4. Litmus test after each extraction: *"If a model reads only the map, can it still avoid
   every known trap?"* — must remain true.
5. **Conservation of facts.** After any reorganization, re-read the old version and confirm
   every technical fact either stayed in the map or landed in a deep dive. Report the check.
6. Update the contract file's decision log with what you changed and why.

## Checklist D — UI/UX decisions & visual changes

Every UI/UX decision — a new screen, a changed color/spacing/copy/flow, or a divergence
from a design prototype — is documented **in the same task** as the change. Full
templates and the token-bootstrap procedure: `references/uiux.md`.

1. **Docs home.** Default structure (the contract file can override names/locations):
   - `docs/uiux/design-tokens.md` — the full token set; the ONLY UI/UX doc where raw
     values (hex, px) may appear.
   - `docs/uiux/<flow>.md` — one file per user flow: Mermaid `stateDiagram-v2` +
     a transitions table (trigger → target → side effects).
   - `docs/uiux/screens/<screenId>.md` — one file per screen, fixed template:
     Purpose / Anatomy (ordered element table) / States / Interactions / Copy /
     Divergences. Screens carry stable slug IDs used consistently everywhere.
2. **Token gate.** The project HAS a token set when a code token module exists AND
   `design-tokens.md` mirrors it (the code is the source of truth; sync both in the same
   task). In that mode, screen/flow docs reference **token names only** — a raw hex or
   size literal outside `design-tokens.md` is a defect. New/reworked UI code must
   consume the token module, never inline literals.
3. **No token set → ask the user ONCE:** does the project want one (code module +
   `design-tokens.md`)? Record the answer with date in the project **contract file**
   (Step 1.3) so future runs never re-ask.
   - Accepted → bootstrap it per `references/uiux.md` (extract from existing
     styles/prototypes; flag near-duplicate values to the user before unifying them).
   - Declined → still document every UI/UX decision, in **simple mode**: one
     `docs/uiux.md` decision log (date · screen · what changed · why · values); raw
     values allowed there.
4. **Divergences are first-class.** When the implementation deviates from a design
   prototype/mockup, record it in the screen doc's Divergences section: what diverged,
   why, and who decided. An undocumented divergence silently turns the prototype into a
   false source of truth.
5. **Copy is spec.** UI strings appear verbatim in the screen doc's Copy column/table;
   changing a string IS a UI/UX decision and updates the doc in the same task.

## Step 3 — Verify (always, before reporting)

Every check below must be an **actually executed** tool call (Grep/Glob/Read/shell) made
NOW, in this session. Never report a number or a "confirmed" you did not read from a tool
result — a fabricated verification is worse than none. Quote the real result for each
check in your report.

1. Every relative link/path referenced in files you touched exists — check each with
   Glob/Read; list any that are missing and fix them.
2. Count the map's lines and state the real number next to its budget. Within ~5% over →
   report it as met and move on. Meaningfully over → ONE whole-block extraction per
   Checklist C item 1; do not phrase-trim your way down.
3. **Stale-phrase sweep:** Grep the map for the OLD behavior's distinctive wording (the
   phrases you replaced). Zero hits expected — any hit is a spot you missed in step A1.
4. Grep the map for chronology markers you may have introduced or missed:
   `SOLVED`, `previously`, `v\d+\.\d+`, `TODO`. Justify or remove each hit in map text
   (deep dives may keep them).
5. If a mirror exists: extract the `##`/`###` headings from both files, count them, and
   confirm they correspond 1:1 in order and numbering — using the real extracted lists,
   not an assumption.
6. If Checklist D ran in token mode: grep the screen/flow docs (everything under the
   UI/UX home except `design-tokens.md`) for raw value literals
   (`#[0-9a-fA-F]{3,8}\b`, `\b\d+(px|dp|pt)\b`) — zero hits expected; and confirm every
   token name those docs reference exists in BOTH `design-tokens.md` and the code token
   module (grep, don't assume).
7. Report: what changed in which file, which checklist items were no-ops, which contract
   file you read in Step 1.3 (or that none exists), and any discrepancies deferred to the
   user.

## Hard limits

- Never invent facts to fill a section — write only what you verified in the code or were
  told; mark genuinely unknown areas as such or ask.
- Never invent design values. Every color/size/copy string in a UI/UX doc is read from
  the code, a design prototype, or the user — and its source is knowable. Two
  near-identical values (e.g. `#1a1a2e` vs `#1a1b2e`) are surfaced to the user before
  being unified, never silently merged.
- Never restructure the map's skeleton, delete a map section, or drop a translation mirror
  without the user's explicit approval.
- If the project contract contradicts this skill, the contract wins — but tell the user
  when the contradiction looks like an error.

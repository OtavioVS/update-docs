# Bootstrap — creating architecture docs in a project that has none

Follow this when Step 1 of the skill found no map file. The output is three things: a
**map**, a **contract file**, and (only if warranted) initial **deep dives**.

## 1. Inventory the project first — write nothing yet

1. Read `README.md`, `CLAUDE.md`, and the top-level folder structure.
2. Identify: entry points, the major runtime components, external systems it talks to
   (databases, APIs, queues, devices), and how it is built/deployed.
3. Hunt for **silent-failure invariants** — the most valuable content a map can hold.
   Grep for: `do not`, `don't`, `must`, `never`, `important`, `workaround`, `hack`,
   `NOTE:`, `WARNING`, pinned/odd version constraints, and config flags whose value looks
   deliberate. Each hit is a candidate rule; verify it in the code before writing it down.
4. Ask the user about anything load-bearing you cannot verify (why a dependency is pinned,
   why a legacy path exists). Do not guess.
5. Note whether the project has **UI surfaces** (screens, components, styles). If it
   does, `uiux.md` §7 applies: ask the user whether to set up the full `docs/uiux/`
   structure or simple mode, and whether to introduce a design-token set; record both
   answers in the contract file. If a company design prototype/mockup exists, record it
   as the canonical design source.
6. **Check the existing docs against the code as you inventory.** In a project with docs
   but no map, the highest-value output of a bootstrap is usually not the new structure —
   it is the contradictions you surface on the way: a rule about a dependency that is no
   longer in the manifest, a status table that never got updated, a stated threshold the
   pipeline does not enforce. Grep the code for the load-bearing claims you read; every
   mismatch is a finding worth more than any amount of reorganizing.

   In an agent-instruction file (`CLAUDE.md` and friends) a stale claim is worse than
   stale prose: it is loaded into every future session, so a wrong rule keeps costing.
   Report those findings explicitly, separately from the structural work.

## 2. Choose the skeleton — sections must earn their place

Archetype skeleton (adapt, don't copy blindly):

1. **The Big Picture** — the one central decision + a small diagram.
2. **The Rules** — silent-failure invariants, one numbered line each (only if you found ≥2).
3. **Infrastructure / external systems** — table (only if there are external systems).
4. **Lifecycles** — app/process states, request or call flows, session/auth lifecycles
   (only for systems with meaningful runtime states).
5. **Layers** — the layer map + an interfaces/modules table.
6. **Platform / environment notes** — per-OS, per-browser, per-deploy-target quirks.
7. **Architecture Decisions** — "why X over Y", one short subsection each.
8. **Dependencies** — table: name, version, what/why.
9. **File structure** — annotated tree.
10. **Key constraints** — build/config checklist table.

Rules of thumb:

- A CLI tool or library may need only sections 1, 5, 7, 8, 9, 10. Do not create empty
  sections to satisfy the archetype.
- Number the sections — cross-references ("§4.1", "Rule 3") are what keep a map coherent.
- Target ≤ ~650 lines from day one (a soft target — see the skill's Checklist C). If a
  section wants to be longer than ~40 lines, its depth belongs in a deep dive as a whole
  block, not trimmed sentence by sentence.

## 3. Write the map header

The first block of the map must state the two-tier model and link the contract file:

```markdown
> **How this documentation is organized.** This file is the **map**: current state only,
> with a pointer to a deep-dive wherever more detail exists. Deep dives live in `<docs
> dir>/`. When a feature lands, update the relevant sections and tables here — do not
> append new narrative sections. Maintenance rules: [<contract file>](<relative path>).
```

## 4. Create the contract file

Place it next to the map (suggested name: `documentation-rules.md`; the user may choose
any name). Whatever the name, **the map header (§3) must link it** — and add a pointer
line in `CLAUDE.md` if the project has one — because that link is how every future skill
run locates the contract, especially under a non-default name. It records the
**project-specific** decisions this skill will read on every future run:

```markdown
# Documentation rules — <project>

- **Map:** <path> — canonical. Skeleton: <list the ## sections chosen and why>.
- **Deep dives:** <docs dir>/, naming convention: <match what the repo already uses>.
- **Budgets:** map ≤ <n> lines; section ≤ ~40 lines body.
- **Last audited:** <commit hash> (<date>) — updated by every audit (skill Checklist E).
- **Mirrors:** <none | path + policy: what stays untranslated, sync-in-same-task rule>.
  A mirror is not always a *translation* mirror: a folder published to a wiki (Azure DevOps,
  Confluence, GitHub wiki) is a **format/publication** mirror. Before reducing a duplicated
  page there to a pointer, check whether repo-relative links actually resolve from the
  published surface — usually they do not, so a pointer strands the reader who only ever sees
  the wiki. Prefer keeping such a page whole, declare it a format mirror with a
  same-task sync rule, and record which map sections map to which page.
- **Deep-dive header format:** Status: CURRENT | PLAN | HISTORICAL + summary pointer.
- **UI/UX docs:** <n/a | simple mode: docs/uiux.md | full: docs/uiux/ + screens/>.
  Token set: <path + enforcement level | declined <date>>. Canonical design source:
  <path/URL of prototype | none>.

## Decision log
- <date>: bootstrapped by update-docs skill. <Any judgment calls made and why.>
```

## 5. Verify

Run the skill's Step 3 verification, plus: confirm with the user that the skeleton and the
rules you inferred (especially anything in a "Rules" section) are actually true — a wrong
invariant in a map is worse than a missing one.

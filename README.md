# update-docs — Claude Code skill

A Claude Code skill that maintains a project's architecture documentation using a
**two-tier system**: one "map" file (the big picture, rules, decisions, constraints —
current state only) plus deep-dive docs (implementation walkthroughs, debugging
narratives, plans — no size limit).

## What it handles

- Folding a just-landed feature or fix into the docs
- Fixing docs that are stale or contradict the code
- Reorganizing an oversized map (whole-block extraction to deep dives, soft line budgets)
- Documenting UI/UX decisions and visual changes, including a design-token workflow
- Bootstrapping architecture docs in a project that has none

## Install

Copy this folder to `~/.claude/skills/update-docs` (or add it to a project's
`.claude/skills/`). Invoke with `/update-docs [what changed, or 'bootstrap' |
'reorganize' | 'audit']`.

## Layout

- `SKILL.md` — the skill definition: doc-location discovery, task classification,
  checklists A–D, verification steps, hard limits
- `references/bootstrap.md` — procedure for creating docs from scratch
- `references/uiux.md` — UI/UX doc templates and the design-token bootstrap

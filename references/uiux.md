# UI/UX documentation — templates & token bootstrap

Read this when Checklist D runs. It defines the file templates, the design-token
bootstrap procedure, the simple (no-token) mode, and the new-project instructions.
The goal of every template: an AI agent (or a developer) can implement the screen
from the doc alone — no Figma, no screenshots — and copy-paste values safely.

## Why written specs (not visual files)

- Token names make intent machine-readable ("`color.danger`" vs "some red").
- Tables and Mermaid diagrams diff cleanly in git and are parsed natively by agents.
- Divergence sections keep a company prototype honest once the implementation moves on.

## 1. Structure

```
docs/uiux/
  design-tokens.md      # the full token set — the ONLY doc holding raw values
  <flow>.md             # one per user flow (e.g. callFlow.md)
  screens/
    <screenId>.md       # one per screen (slug id, e.g. activeCall.md)
```

Screens get **stable slug IDs** (`s-transfer`, `activeCall`) used in docs, code
comments, and conversation. Never rename an ID silently; that breaks addressability.

## 2. design-tokens.md template

Header states the paired code module and the sync rule:

```markdown
# Design tokens
**Status:** CURRENT
**Code module:** `<path, e.g. src/theme/tokens.ts>` — source of truth; this file mirrors
it and MUST be updated in the same task as any token change.

## Colors
| Token | Value | Used for |
|---|---|---|
| `color.action` | `#1e9cd7` | primary action buttons, active icons |
…

## Typography  (family, then a size/weight scale table)
## Radii · Spacing · Sizing  (scale tables)
## Naming rules  (how new tokens are named; when to add vs reuse)
```

Rules:
- Semantic names over raw-scale names where the meaning is stable (`color.danger`,
  not `red500`); scale names are fine for graded families (`blue100…blue900`).
- A value used once may stay a literal in `design-tokens.md`'s notes; a value used
  twice gets a token.
- Deleting/renaming a token: grep the code AND all screen docs for the old name first.

## 3. Screen doc template

```markdown
# <Human name> (`<screenId>`)
**Status:** CURRENT | PLAN
**Owned by:** <native | web | …>   ← who renders it, if the project mixes surfaces
**Entered from:** <screen/trigger> | **Exits to:** <screens>

<one-paragraph purpose>

## Anatomy (top → bottom)
| # | Element | Spec (token references only) | Copy |
|---|---------|------------------------------|------|
| 1 | … | bg `color.surfaceDark`, radius `radius.full`, 52×52 | "Transferir" |

## States
One bullet per visual state (empty / loading / disabled / error / …): what triggers
it and what changes, referencing anatomy row numbers.

## Interactions
| Trigger | Result | Side effects |
|---------|--------|--------------|

## Copy
| Key/context | String (verbatim, target language) |
|---|---|

## Divergences from <prototype/design source>
| What | Prototype says | We do | Why / decided by |
|---|---|---|---|
```

- Sizes in the Anatomy column are plain numbers matching the platform's unit (RN dp,
  web px) — state the unit once at the top of `design-tokens.md`.
- If a screen exists in the app but not in the design source (or vice versa), it still
  gets a doc; the Divergences section says "not in prototype" / "not implemented" and why.

## 4. Flow doc template

```markdown
# <Flow name> — flow spec
**Status:** CURRENT
**Screens:** links to every screens/<id>.md involved

​```mermaid
stateDiagram-v2
    [*] --> screenA
    screenA --> screenB: trigger
​```

## Transitions
| From | Trigger | To | Side effects (timers, banners, notifications) |
|---|---|---|---|

## Divergences from <design source>  (same table shape as screens)
```

## 5. Bootstrapping a token set (user accepted the ask)

1. **Harvest** every raw value: grep the UI code for `#[0-9a-fA-F]{3,8}\b` and
   `rgba?\(`, plus the design prototype's values if one exists. List them with
   file:line provenance.
2. **Cluster near-duplicates** (values within a hair of each other, e.g. `#1a1a2e` vs
   `#1a1b2e`, two different "danger" reds). These are usually drift, not intent —
   present the clusters to the user and let them pick the canonical value BEFORE
   writing the token. (If the user already declared a design source canonical, its
   value wins; still report the unifications made.)
3. **Name** tokens per §2's rules; write the code module first, then
   `design-tokens.md` mirroring it.
4. **Refactor** existing UI code to consume the module. Colors are mandatory;
   size/spacing scales may be introduced as guidance first and enforced for new code —
   record which level was chosen in the contract file.
5. **Record in the contract file:** date, token module path, enforcement level,
   canonical design source. Then run Step 3 verification including the raw-value sweep.

## 6. Simple mode (user declined a token set)

One file, `docs/uiux.md`, append-per-decision (this is the ONE place chronological
append is correct — it is a log, not a map):

```markdown
# UI/UX decision log
| Date | Screen/flow | What changed | Why | Values touched |
|---|---|---|---|---|
```

Raw values allowed. If the log grows past ~150 rows or the team starts asking "what's
our blue?", suggest upgrading to the full structure — ask the user again only then
(and update the contract with the new answer).

## 7. New projects (bootstrap runs)

During `references/bootstrap.md` §1 inventory, note whether the project has UI
surfaces at all. If it does:

- Ask the user two questions alongside the skeleton confirmation: (a) full
  `docs/uiux/` structure or simple mode? (b) introduce a token set? Record both in the
  new contract file (one line each + decision-log entry).
- If a company design prototype/mockup exists, name it in the contract as the
  **canonical design source** — Divergences sections are measured against it.
- A project with no UI (CLI, library, service) records "UI/UX docs: n/a" in the
  contract so future runs skip Checklist D instantly.

## 8. What is NOT a divergence

The Divergences section is the highest-signal part of a screen doc, and the easiest to dilute.
It holds **implementation that departs from a design/spec source**, or **behavior that contradicts
what the screen itself promises**. Nothing else — padding it buries the deviation that matters.

| Belongs in Divergences | Does NOT — and where it goes instead |
|---|---|
| Implementation differs from the prototype/mockup | A choice the plan explicitly **sanctioned** ("put it in A or B") — that is a decision, not a deviation |
| Behavior contradicts the state the screen advertises | Sequencing between plan phases/waves → the plan doc |
| Copy differs from what was specified | An architecture decision → the map |
| A known limitation the user actually hits | A feature never promised → the backlog, if anywhere |

When nothing departs, write **"None"**. An empty Divergences section is a valid, informative result —
far better than three entries that are really decisions.

> Why this rule exists: in a real run, a screen doc got three "divergences". Two were sanctioned
> plan decisions, and the third was a misreading of the error path. The wrong entry then masked an
> actual defect on that same code path for a full round of review — the padding did not just add
> noise, it hid the finding.

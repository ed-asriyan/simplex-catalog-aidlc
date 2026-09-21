# Refined Mockups — Questions

## Sources

- User stories `../user-stories/stories.md`
- Requirements `../requirements-analysis/requirements.md`
- Team practices `../practices-discovery/team-practices.md`
- Existing UI inspected: `src/components/servers/table/index.svelte` (the filter
  card + toolbar), `src/components/servers/list.svelte` (filter/sort/URL wiring).
  The existing Location filter already surfaces `TOR`, `YGGDRASIL`, `I2P` as
  special values, so the tor/clearnet presets map directly onto it.

Note: rough-mockups/wireframes/user-flow are absent by scope design (no Ideation
rough-mockups stage); mockups are designed directly from stories + requirements.

## Q1. Where should the quick-filter bar (presets + custom filters) go?

- A. A dedicated horizontal bar of chips/buttons ABOVE the existing filter card,
  so a click sets the filter card + sort below it (recommended — keeps the quick
  filters prominent and the detailed controls as the "expanded" form)
- B. Inside the existing filter card (e.g. a top row within it)
- C. Somewhere else
- X. Other (please specify)

[Answer]: A. Dedicated horizontal chip/button bar ABOVE the existing filter card; clicking a preset/custom chip sets the filter card + sort below.
## Q2. How are custom-filter management actions (rename / edit-in-place / delete) exposed?

- A. Each custom-filter chip has a small "⋯" menu (UIkit dropdown) with Rename /
  Update to current view / Delete; clicking the chip body applies it
  (recommended)
- B. A separate "Manage filters" dialog listing all custom filters with actions
- C. Inline icons on each chip (pencil / save / trash)
- X. Other (please specify)

[Answer]: A. Each custom-filter chip has a small ⋯ menu (UIkit dropdown) with Rename / Update to current view / Delete; clicking the chip body applies it.
## Q3. Save / rename / delete dialogs — native or UIkit modal?

The app today uses native `prompt()`/`confirm()` (e.g. Add server, Import labels).

- A. Match the existing app: native `prompt()` for name entry (save/rename) and
  native `confirm()` for delete (recommended — consistent, minimal, matches the
  codebase; delete confirm satisfies AC2.3.4)
- B. Use UIkit modals (`uk-modal`) for a more polished look (more markup/work)
- X. Other (please specify)

[Answer]: A. Match the existing app — native prompt() for name entry (save/rename) and native confirm() for delete (satisfies AC2.3.4).
## Q4. Accessibility target level?

- A. WCAG 2.1 AA (recommended — standard bar; matches NFR4 keyboard + AT state)
- B. WCAG 2.1 AAA
- X. Other (please specify)

[Answer]: A. WCAG 2.1 AA.
## Q5. Selected-state visual treatment for the active preset/custom chip?

- A. Active chip uses UIkit's primary/active button style (`uk-button-primary`)
  with `aria-pressed="true"`; others use default style (recommended)
- B. Underline/checkmark indicator
- X. Other (please specify)

[Answer]: A. Active chip uses uk-button-primary with aria-pressed="true"; inactive chips use uk-button-default with aria-pressed="false".

## Consolidated Summary Confirmation

The refined-mockups artifacts resolve to: a **quick-filter bar above the existing
filter card** with 5 preset chips + saved custom chips + a "+ Save current"
control; **per-chip ⋯ menu** (Rename… / Update to current view / Delete);
**native prompt()/confirm()** dialogs (delete confirmed); active chip =
`uk-button-primary` + `aria-pressed`; **WCAG 2.1 AA**. Four artifacts produced —
`mockups.md` (11 textual wireframes incl. apply, empty result, save, rename,
update-in-place, delete-confirm, modified-view deselect, storage-unavailable,
default-load, tie-precedence), `interaction-spec.md` (component specs + a single
`applyView` helper for atomic filter+sort URL writes per AC1.1.6),
`design-system-mapping.md` (UIkit mapping + preset→Filter/Sort table: clearnet =
exclusive [TOR,I2P,YGGDRASIL], tor = inclusive [TOR], uptime90 = 0.9 fraction),
and `accessibility-checklist.md` (AA + NFR4). Advisory product-lead review: READY;
its 1 Major (atomic apply → applyView helper) and 4 Minor findings were all
applied; requirements AOQ-5 wording closed. No backend change. Revision: the
layout now draws EVERY section (Header/intro, Quick filters, Filters card,
Toolbar, Table) as a separate bordered box, and the Quick-filters bar is its own
distinct `uk-card` sibling container (never nested in the header or filter card),
so future edits can't accidentally merge sections.

[Answer]:

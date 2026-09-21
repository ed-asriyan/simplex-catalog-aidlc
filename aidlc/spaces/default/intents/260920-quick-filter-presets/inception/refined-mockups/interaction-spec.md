# Interaction Specification — Quick-Filter Presets & Custom Filters

Component-level specs (per `.claude/knowledge/aidlc-design-agent/component-spec-template.md`).
Behaviour traces to `../user-stories/stories.md` and `../requirements-analysis/requirements.md`.

## Sources

- User stories `../user-stories/stories.md`
- Requirements `../requirements-analysis/requirements.md`
- Team practices `../practices-discovery/team-practices.md` (feature layering, index.svelte entry)
- Existing wiring `src/components/servers/list.svelte`, `src/components/servers/table/index.svelte`

## Assumptions & Open Questions

None.

## Component tree (proposed, per team layering)

```
components/servers/quick-filters/
  index.svelte            ← the bar (presets + custom chips + save control)
  preset-chip.svelte      ← one preset button
  custom-chip.svelte      ← one custom chip + ⋯ dropdown menu
store/servers/quick-filters/
  presets.ts              ← hardcoded preset definitions (filter + sort)
  custom-filters-store.ts ← localStorage-backed CRUD + state
  quick-filters-service.ts← apply/match/serialize helpers (view = filter + sort)
  analytics.ts usage      ← emit apply events (reuse existing analytics)
```
The bar mounts in `list.svelte` above the `<ServersTable>` filter card. To
satisfy the atomic-apply criterion (AC1.1.6), applying a preset/custom view must
be ONE coherent update, not two. Today `updateFilter()` and `updateSort()` each
call `setQueryParam` separately, i.e. two URL writes. The store layer therefore
exposes a single **`applyView({ filter, sort })`** helper that writes the filter
params AND `sortField`/`sortOrder` in ONE `setQueryParam` call (and one
last-used-localStorage write), so the URL reflects both at once with no
intermediate single-changed state. `list.svelte` passes `applyView` to the bar;
ad-hoc control edits keep using the existing `updateFilter`/`updateSort`.

---

## quick-filters bar  (index.svelte)

| Field | Value |
|---|---|
| Component | quick-filters bar |
| Description | Row of preset buttons + saved custom chips + "Save current" control |
| Category | navigation |

### States

| State | Description | Trigger |
|---|---|---|
| default | Presets + any custom chips render; the chip matching the active view is selected | page load / view change |
| none-selected | Active view matches no chip (ad-hoc) — no chip highlighted | user edits filter/sort (US1.3/US2.6) |
| empty-custom | No saved custom filters — presets + "Save current" only | first visit / storage empty (AC2.1.6) |
| storage-unavailable | localStorage unreadable/malformed — presets only, no error | AC3.1.3 / NFR5 |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| filter | Filter | yes | — | current active filter (from list.svelte) |
| sort | Sort | yes | — | current active sort |
| applyView | ({filter, sort}) => void | yes | — | single-write helper: sets filter params + sort params in ONE URL update (AC1.1.6); used for preset/custom apply |
| updateFilter | (Filter) => void | yes | — | existing setter, used only for ad-hoc control edits |
| updateSort | (Sort) => void | yes | — | existing sort setter, used only for ad-hoc control edits |

### Behaviour

- Renders `presets` (from `presets.ts`) then custom chips (from
  `custom-filters-store`), then the `+ Save current` control.
- Computes the selected chip by matching `{filter, sort}` against each preset
  then each custom definition; preset wins ties (AC5.1.4/AC5.1.5). Match is
  order-insensitive on multi-value facets (e.g. `countries.values`).

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | Native `<button>` per chip; the bar is a `<div role="group" aria-label="Quick filters">` |
| Keyboard | Tab to each chip; Enter/Space activates (native button) |
| Label | Visible text label; the ⋯ trigger has `aria-label="Manage <name>"` |
| Contrast | WCAG AA (UIkit primary/default already AA) |
| Screen reader | Selected chip exposes `aria-pressed="true"` (AC5.1.9) |

---

## preset-chip  (preset-chip.svelte)

| Field | Value |
|---|---|
| Component | preset-chip |
| Description | One hardcoded preset button that applies its view on click |
| Category | navigation |

### States

| State | Description | Trigger |
|---|---|---|
| default | uk-button-default, aria-pressed=false | not the active view |
| selected | uk-button-primary, aria-pressed=true | active view equals this preset (AC5.1.1) |
| hover / focus | UIkit hover + visible focus ring | mouseover / Tab |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| preset | { id, label, filter, sort } | yes | — | hardcoded definition |
| selected | boolean | yes | false | whether it matches the active view |
| onApply | (preset) => void | yes | — | replaces the whole view + emits analytics |

### Behaviour

- Click → `onApply(preset)` → parent calls `applyView({ filter: preset.filter,
  sort: preset.sort })` (whole-view replace in ONE URL write, AC1.1.3/AC1.1.4/
  AC1.1.6), then emits a preset-apply analytics event (AC4.1.1). Empty result set
  is allowed (AC1.1.5).

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | button |
| Keyboard | Enter/Space activate |
| Label | preset label text |
| Screen reader | announces label + pressed state |

---

## custom-chip  (custom-chip.svelte)

| Field | Value |
|---|---|
| Component | custom-chip |
| Description | Saved custom filter: body applies it; ⋯ menu manages it |
| Category | navigation |

### States

| State | Description | Trigger |
|---|---|---|
| default | uk-button-default with trailing ⋯; aria-pressed=false | not active |
| selected | uk-button-primary, aria-pressed=true | active view equals its saved def (AC5.1.2) |
| menu-open | ⋯ dropdown visible (Rename… / Update to current view / Delete) | click ⋯ |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| filter | { id, name, filter, sort } | yes | — | saved custom filter (stable `id`, AC2.1.4) |
| selected | boolean | yes | false | matches active view |
| onApply | (cf) => void | yes | — | apply saved view + analytics (AC2.2.1, AC4.1.2) |
| onRename | (cf) => void | yes | — | prompt() new name (AC2.4.x) |
| onUpdate | (cf) => void | yes | — | re-save current view over it (AC2.5.x) |
| onDelete | (cf) => void | yes | — | confirm() then remove (AC2.3.x) |

### Behaviour

- Click chip **body** → `onApply` (whole-view replace; emits custom-apply event
  with the stable `id`, AC4.1.2).
- Click **⋯** → open menu; menu actions do NOT apply the filter (AC2.3.3).
  - Rename… → `prompt("Rename filter", currentName)`; empty/whitespace rejected
    (AC2.4.3); id + definition unchanged (AC2.4.1).
  - Update to current view → replaces the saved definition with the current
    `{filter, sort}`; keeps name + id (AC2.5.1).
  - Delete → `confirm("Delete filter \"<name>\"? …")`; on OK remove (AC2.3.4).

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | button (body) + button (⋯ trigger) + UIkit dropdown menu |
| Keyboard | Tab to chip and to ⋯; Enter/Space activate; Esc closes menu; focus returns to ⋯ on close |
| Label | chip name; ⋯ has aria-label; menu items are real buttons/links |
| Screen reader | selected state via aria-pressed; menu announced as menu |

---

## save-current control

| Field | Value |
|---|---|
| Component | save-current button |
| Description | Saves the current view as a new named custom filter |
| Category | input |

### Behaviour

- Always visible in the bar (discoverability, AC2.1.6).
- Click → `prompt("Name this filter")`. Non-empty → create custom filter with a
  new stable id capturing the current `{filter, sort}` (AC2.1.1, AC2.1.3,
  AC2.1.4); it becomes the selected chip. Empty/whitespace or Cancel → no-op
  (AC2.1.5).

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | button |
| Keyboard | Enter/Space |
| Label | Visible text `+ Save current`; accessible name `Save current filter` (via `aria-label`, since the visible "+" glyph is decorative) |

---

## Cross-cutting interaction rules

- **View = filter + sort**, applied together via `applyView` in ONE URL write
  (AC1.1.6); the same query params + last-used localStorage update as today
  (AC3.2.1). Preset/custom name is never written to the URL (AC3.2.3).
- **Ad-hoc edit** (changing a filter/sort control while a chip is active):
  handled by the existing controls; the bar simply re-derives selection →
  deselects (US1.3/US2.6). No analytics apply event (AC4.1.5).
- **Analytics** reuse the existing measurement id; safe no-op when unconfigured
  (AC4.1.3); identifier never added to the URL (AC4.1.4).
- **Persistence** via a dedicated localStorage key, separate from
  `serversFilter` (AC3.1.2); all reads/writes wrapped in try/catch (AC3.1.3).

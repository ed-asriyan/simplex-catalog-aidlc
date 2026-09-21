# Design System Mapping — UIkit

Maps each mockup element to the existing UIkit component vocabulary already used
across the servers page, so the feature is visually native and adds no new CSS
framework. Traces to `mockups.md` and `interaction-spec.md`.

## Sources

- Existing UIkit usage: `src/components/servers/table/index.svelte`
  (uk-card, uk-button, uk-button-small, uk-select, uk-dropdown, uk-grid, uk-flex)
- Team practices `../practices-discovery/team-practices.md` (UIkit is the design system)
- Requirements `../requirements-analysis/requirements.md`

## Assumptions & Open Questions

None.

## Element → UIkit mapping

| Mockup element | UIkit classes / attributes | Notes |
|---|---|---|
| Quick-filter bar container | `uk-flex uk-flex-wrap uk-flex-middle` with inline `gap: 8px`, `uk-margin-small-bottom`; `role="group" aria-label="Quick filters"` | Mirrors the existing toolbar row's flex-wrap+gap pattern in table/index.svelte |
| Section within existing filter card | (unchanged) `uk-card uk-card-default uk-card-body uk-card-small` | Bar sits ABOVE this card, not inside |
| Preset chip (inactive) | `uk-button uk-button-default uk-button-small` + `aria-pressed="false"` | Matches existing small buttons |
| Preset / custom chip (active) | `uk-button uk-button-primary uk-button-small` + `aria-pressed="true"` | Selected-state treatment (Q5=A) |
| Custom chip body | `uk-button uk-button-default uk-button-small` (or primary when active) | Click = apply |
| Custom chip ⋯ trigger | small `uk-button` / icon-only button with `aria-label="Manage <name>"` + `uk-toggle`/`uk-dropdown` sibling | Does not apply the filter |
| ⋯ menu | `<div uk-dropdown="mode: click">` containing `uk-nav uk-dropdown-nav` with items Rename… / Update to current view / Delete | Same dropdown component the Location/Labels filters use |
| Save current control | `uk-button uk-button-default uk-button-small` labelled `+ Save current` | Always present (discoverability) |
| Name entry (save/rename) | native `window.prompt()` | Matches Add-server / Import-labels pattern (Q3=A) |
| Delete confirm | native `window.confirm()` | AC2.3.4 |
| Loading (table) | existing `uk-spinner` in table/index.svelte | Unchanged |
| Empty result | existing "Total servers matching filters: 0" block | Unchanged (AC1.1.5) |
| Icon (⋯ / +) | existing `Icon` component (`src/components/icon.svelte`) or a text glyph | Reuse app icon convention |

## Preset → existing Filter/Sort field mapping

| Preset | Filter (existing `Filter` fields) | Sort (existing `Sort`) |
|---|---|---|
| All Online | `{ status: true }` | `{ field: 'last_check', order: 'desc' }` |
| Online Clearnet | `{ status: true, countries: { inclusive: false, values: ['TOR','I2P','YGGDRASIL'] } }` | `{ field: 'last_check', order: 'desc' }` |
| Online Tor | `{ status: true, countries: { inclusive: true, values: ['TOR'] } }` | `{ field: 'last_check', order: 'desc' }` |
| Recently Added | `{ status: true }` | `{ field: 'created_at', order: 'desc' }` |
| High Uptime | `{ status: true, uptime90: 0.9 }` | `{ field: 'uptime90', order: 'desc' }` |

Notes:
- `uptime90` in the `Filter` is a 0–1 fraction (the table UI multiplies by 100
  for display and divides on input), so 90% ⇒ `0.9`. Confirmed against
  `table/index.svelte` (`filter.uptime90 * 100` / `+e.target.value / 100`).
- The Location filter already lists `TOR`, `YGGDRASIL`, `I2P` as special values
  (`countriesSorted` special array), so these presets reuse the exact same
  mechanism as the manual Location control.
- `status: true` renders as the existing Status = "Active" option.

## Responsive mapping

| Breakpoint | Behaviour |
|---|---|
| mobile (<768px) | Bar wraps to multiple lines (`uk-flex-wrap`); chips remain ≥32px tall tap targets; no horizontal scroll |
| tablet (768–1024px) | Chips flow on 1–2 lines; filter card uses its existing `uk-child-width-1-3@m` grid |
| desktop (>1024px) | Single-line bar typical; filter card `uk-child-width-1-4@l` (unchanged) |

## Theming

- No new colors; UIkit primary is used only for the active chip. Dark/light
  themes inherit from the existing UIkit theme already shipped by the app.

# Requirements — Servers Quick-Filter Presets & Custom Filters

Feature: quick-filter preset buttons plus user-defined custom filters on the
servers page of `simplex-catalog-frontend`, replacing today's
enter-filters-every-time workflow.

## Sources

- Intent statement: `ideation/intent-capture/intent-statement.md` [desc]
- Requirements interview answers (Q1–Q8) and the follow-up refinement:
  `requirements-analysis-questions.md`
- Existing filter/sort model: `simplex-catalog-frontend/src/store/servers/servers-service.ts`
  (the `Filter`, `Sort`, and `Server` types; the client-side filtering block at
  lines 183–197; `countries` is filtered client-side with an inclusive/exclusive
  `FilterArray`) [scope]
- Existing filter/sort application & persistence: `simplex-catalog-frontend/src/components/servers/list.svelte`
  (`updateFilter()` writes both URL query params and `localStorage['serversFilter']`;
  `updateSort()` writes `sortField`/`sortOrder` query params; `sort` defaults to
  `last_check`/`desc`; `defaultFilter = { status: true }`) [scope]
- Location/overlay-network markers: `simplex-catalog-frontend/src/utils.ts`
  (`getFlagEmoji` special-cases `TOR`, `I2P`, `YGGDRASIL` location codes) [scope]
- Team practices: `inception/practices-discovery/team-practices.md`

## Glossary

- **Preset** — a hardcoded, read-only quick-filter button that applies a
  predefined *view* (a filter **and** a sort) on click.
- **Custom filter** — a user-created, named, editable button that applies a
  saved view (filter + sort), persisted in the browser.
- **Ad-hoc filter** — a filter/sort the user sets through the normal controls
  without saving it as a custom filter.
- **View** — the combination of a `Filter` and a `Sort`. This is exactly what
  the page URL already encodes (filter query params + `sortField`/`sortOrder`).
- **location** — the server's `country` field. For servers reachable only over
  an overlay network it carries a network marker string — `TOR`, `I2P`, or
  `YGGDRASIL` — instead of a geographic 2-letter country code.
- **clearnet** — a server whose location is a real geographic country code,
  i.e. location is NOT one of the overlay markers `TOR`, `I2P`, `YGGDRASIL`.
- **tor** — a server whose location is `TOR`.

## Actors

- **Catalog visitor** — any user of the servers page; applies presets and
  ad-hoc filters, and creates/manages custom filters. No authentication; all
  state is per-browser.

## Functional Requirements

### Presets

- **FR-1** The servers page SHALL display a row of quick-filter preset buttons.
  Clicking a preset applies its predefined view immediately, without any further
  confirmation. (Traces: intent — "click one to instantly apply a predefined
  filter".)
- **FR-2** Applying a preset SHALL REPLACE the entire active view — both the
  filter and the sort (any preset, custom, or ad-hoc view currently in effect is
  discarded and replaced). Presets are mutually exclusive: at most one preset is
  the active view at a time. (Traces: Q1=A.)
- **FR-3** Each preset defines a **filter and a sort**. All five presets filter
  to online servers (`status = true`); they differ by an additional filter
  facet and/or their sort. The following five presets SHALL ship, hardcoded:

  | Preset | Filter | Sort |
  |---|---|---|
  | **FR-3.1 All Online** | `status = true` | `last_check` desc |
  | **FR-3.2 Online Clearnet** | `status = true` AND location is clearnet (`countries` = `{ inclusive: false, values: ['TOR','I2P','YGGDRASIL'] }`) | `last_check` desc |
  | **FR-3.3 Online Tor** | `status = true` AND location = `TOR` (`countries` = `{ inclusive: true, values: ['TOR'] }`) | `last_check` desc |
  | **FR-3.4 Recently Added** | `status = true` | `created_at` desc |
  | **FR-3.5 High Uptime** | `status = true` AND `uptime90 ≥ 90` | `uptime90` desc |

  (Traces: Q2 and the follow-up refinement making sort an explicit part of every
  preset and status=online the common base.)
- **FR-4** Every preset SHALL be expressible entirely with the existing
  `Filter` and `Sort` model — including the existing inclusive/exclusive
  `countries` `FilterArray` for the tor/clearnet distinction via the location
  markers. No new filter field, no derived "network" concept, and no
  server-side query or database schema change is required.
- **FR-5** The currently active preset (if any) SHALL be visually indicated as
  selected. Selection is DERIVED by matching the active view (filter + sort)
  against each preset definition — it is not stored. When the active view
  matches no preset (e.g. the user edited it), no preset is shown as selected.

### Custom filters

- **FR-6** A user SHALL be able to save the current view as a named custom
  filter: they configure the normal filter/sort controls, choose "save current
  filter", and provide a name; the saved view then appears as a button alongside
  the presets. A saved custom filter SHALL capture both the filter and the sort,
  consistent with presets. (Traces: Q4=A; refinement that sort is part of a
  view.)
- **FR-7** A user SHALL be able to manage custom filters with full CRUD, with no
  hard limit on how many are saved:
  - **FR-7.1** Create (per FR-6).
  - **FR-7.2** Rename an existing custom filter.
  - **FR-7.3** Edit in place — re-save the current view over an existing custom
    filter, updating its stored definition.
  - **FR-7.4** Delete a custom filter.
  (Traces: Q5=A.)
- **FR-7.5** Clicking a custom filter SHALL apply it with the same
  replace-the-whole-view semantics as a preset (FR-2).

- **FR-8** *(Withdrawn.)* An earlier draft proposed a derived `network`
  classification. Removed: the tor/clearnet distinction is expressed directly
  from the location field via the existing `countries` filter (FR-3, FR-4), so
  no `network` term is defined. ID retained as withdrawn to keep later IDs
  stable.

### Persistence & URL

- **FR-9** Custom filters SHALL be persisted in `localStorage`, per-browser,
  not synced across devices or shared. (Traces: Q6=A.) The stored payload MUST
  be independent of the existing `serversFilter` "last used" key so saved
  filters and the last-used view do not clobber each other.
- **FR-10** Applying any preset, custom filter, or ad-hoc filter SHALL update
  the page URL to reflect the resulting **view only** — the filter query params
  and the `sortField`/`sortOrder` params — exactly as manual filter/sort changes
  do today. The URL SHALL NOT carry any preset id/name or custom-filter
  id/name: a button is purely a shortcut that sets the underlying filter+sort,
  and does not change how state integrates with the URL. Opening such a URL
  SHALL restore that view, keeping every view linkable and bookmarkable.
  (Traces: Q7=A; refinement on URL semantics.)
- **FR-11** The existing default view (`status = true`, sort `last_check`/`desc`)
  SHALL be preserved when no view is supplied by URL, and the feature SHALL be
  purely additive to the current filter/sort mechanics — no existing capability
  is removed.

### Analytics

- **FR-12** On every preset apply and every custom-filter apply, the system
  SHALL emit a lightweight analytics event via the existing
  Google-Analytics-style measurement id (`VITE_ANALYTICS_MEASHUREMENT_ID`),
  carrying an identifier of which preset or custom filter was applied, so
  post-release usage can be measured. (Traces: Q8=A, and the intent's metrics
  goal.) When the measurement id is not configured, emission SHALL be a safe
  no-op (no error surfaced to the user). Note: this analytics identifier is an
  in-event property only and is NOT written to the URL (see FR-10).

## Non-Functional Requirements

- **NFR-1 (Layering)** Filter-preset and custom-filter logic (definitions,
  persistence, apply/serialize, active-preset matching) SHALL live in the store
  layer under `src/store/servers/`, not in component-local logic; presentation
  lives under `src/components/servers/`. Database `snake_case` field names MUST
  NOT leak past the service boundary. (Traces: team Code Style practices.)
- **NFR-2 (Testing)** All new/changed logic SHALL be developed test-first (TDD)
  with Vitest; UI with `@testing-library/svelte`. New/changed code SHALL meet
  the 80% line-coverage floor. CI MUST fail the PR if tests or `svelte-check` do
  not pass.
- **NFR-3 (No backend change)** The feature SHALL NOT require any Supabase
  schema, view, or query change; all preset/custom-filter behaviour is achieved
  with the existing data, filter/sort model, and client-side derivation.
- **NFR-4 (Accessibility)** Preset and custom-filter controls SHALL be
  keyboard-operable and expose their selected/active state to assistive
  technology (consistent with existing UIkit usage).
- **NFR-5 (Graceful storage failure)** If `localStorage` is unavailable or
  malformed, the page SHALL still render and function with presets only;
  custom-filter reads/writes MUST fail safe without breaking the servers list.

## Assumptions & Open Questions

- **AOQ-1** *(Resolved.)* "Recently Added" is a view = `status = true` + sort
  `created_at` desc; no recency-window filter. Confirmed by the refinement that
  sort is an explicit part of every preset.
- **AOQ-2** *(Resolved.)* Tor/clearnet is taken directly from the location
  field's overlay markers (`TOR`/`I2P`/`YGGDRASIL`) via the existing `countries`
  filter; no `.onion` heuristic and no misclassification of un-geolocated
  servers, because the marker is authoritative.
- **AOQ-3** Custom-filter names are assumed free-text; whether duplicate names
  are blocked is a design detail deferred to mockups.
- **AOQ-4** "Clearnet" excludes all three overlay markers (`TOR`, `I2P`,
  `YGGDRASIL`) for technical correctness; the user named Tor and I2P explicitly,
  and `YGGDRASIL` is included as the third overlay marker present in the code.
  Flag if clearnet should instead exclude only `TOR`/`I2P`.
- **AOQ-5** "High Uptime" is specified as `uptime90 ≥ 90` (filter) AND sort by
  `uptime90` desc. If the intent was sort-only (highest-uptime-first with no
  threshold), drop the `≥ 90` filter. To confirm.

## Traceability

| Requirement | Origin |
|---|---|
| FR-1, FR-2, FR-5 | Intent ("instantly apply a predefined filter"); Q1; refinement |
| FR-3 | Q2 + refinement (per-preset filter & sort, status=online base) |
| FR-4 | Existing `Filter`/`Sort`/`countries` model; refinement (location field) |
| FR-6, FR-7 | Intent ("create, save, edit, and persist custom filters"); Q4, Q5 |
| FR-8 | Withdrawn (superseded by FR-3/FR-4) |
| FR-9 | Intent ("persist … in localStorage"); Q6 |
| FR-10 | Existing URL-encoded filter+sort behaviour; Q7; refinement (URL = view only) |
| FR-12 | Intent (measure usage); Q8 |
| NFR-1..NFR-5 | Team practices (layering, TDD/coverage, no-backend-change scope) |

# Requirements — Servers Quick-Filter Presets & Custom Filters

Feature: quick-filter preset buttons plus user-defined custom filters on the
servers page of `simplex-catalog-frontend`, replacing today's
enter-filters-every-time workflow.

## Sources

- Intent statement: `ideation/intent-capture/intent-statement.md` [desc]
- Requirements interview answers (Q1–Q8): `requirements-analysis-questions.md`
- Existing filter model: `simplex-catalog-frontend/src/store/servers/servers-service.ts`
  (the `Filter` and `Server` types, and the client-side filtering block at
  lines 183–197) [scope]
- Existing filter application / persistence: `simplex-catalog-frontend/src/components/servers/list.svelte`
  (`updateFilter()` writes both URL query params and `localStorage['serversFilter']`;
  `defaultFilter = { status: true }`) [scope]
- Team practices: `inception/practices-discovery/team-practices.md`

## Glossary

- **Preset** — a hardcoded, read-only quick-filter button that applies a
  predefined filter view on click.
- **Custom filter** — a user-created, named, editable filter saved in the
  browser and shown as a button alongside the presets.
- **Ad-hoc filter** — a filter the user sets through the normal controls
  without saving it as a custom filter.
- **network** — a derived per-server classification, `tor` or `clearnet`,
  computed client-side (see FR-8), not a stored database column.

## Actors

- **Catalog visitor** — any user of the servers page; applies presets and
  ad-hoc filters, and creates/manages custom filters. No authentication; all
  state is per-browser.

## Functional Requirements

### Presets

- **FR-1** The servers page SHALL display a row of quick-filter preset buttons.
  Clicking a preset applies its predefined filter view immediately, without any
  further confirmation. (Traces: intent — "click one to instantly apply a
  predefined filter".)
- **FR-2** Applying a preset SHALL REPLACE the entire active filter (any preset,
  custom, or ad-hoc filter currently in effect is discarded and replaced by the
  preset's filter). Presets are mutually exclusive: at most one preset is the
  active view at a time. (Traces: Q1=A.)
- **FR-3** The following five presets SHALL ship, hardcoded:
  - **FR-3.1 All Online** — `status = up` (all reachable servers).
  - **FR-3.2 Online Clearnet** — `status = up` AND `network = clearnet`.
  - **FR-3.3 Online Tor** — `status = up` AND `network = tor`.
  - **FR-3.4 Recently Added** — servers ordered most-recently-added first
    (sort by `Server.createdAt` descending). A recency-window filter is NOT
    required for this intent; if design later adds one it MUST reuse
    `createdAt`. (See AOQ-1.)
  - **FR-3.5 High Uptime** — `uptime90 ≥ 90` (%).
  (Traces: Q2.)
- **FR-4** The preset definitions SHALL be expressible entirely with the
  existing `Filter`/`Sort` model plus the derived `network` classification of
  FR-8; no server-side query or database schema change is required.
- **FR-5** The currently active preset (if any) SHALL be visually indicated as
  selected; when the active filter no longer matches any preset (e.g. the user
  edited it), no preset is shown as selected.

### Custom filters

- **FR-6** A user SHALL be able to save the current filter as a named custom
  filter: they configure the normal filter controls, choose "save current
  filter", and provide a name; the saved filter then appears as a button
  alongside the presets. (Traces: Q4=A.)
- **FR-7** A user SHALL be able to manage custom filters with full CRUD:
  - **FR-7.1** Create (per FR-6).
  - **FR-7.2** Rename an existing custom filter.
  - **FR-7.3** Edit in place — re-save the current filter over an existing
    custom filter, updating its stored definition.
  - **FR-7.4** Delete a custom filter.
  There SHALL be no hard limit on the number of saved custom filters.
  (Traces: Q5=A.)
- **FR-7.5** Clicking a custom filter SHALL apply it with the same
  replace-the-whole-filter semantics as a preset (FR-2).

### Derived tor/clearnet classification

- **FR-8** The system SHALL derive each server's `network` classification
  client-side, without a backend/query change (mirroring how `status`,
  `protocol`, and `country` are already filtered in JS in
  `servers-service.ts`):
  - **FR-8.1** Primary signal (per Q3): a server whose location/`country` is
    empty is classified `tor`; a server with a known `country` is `clearnet`.
  - **FR-8.2** The host `.onion` suffix is the authoritative signal that an
    address is a tor address; design SHALL reconcile FR-8.1 with the
    host-`.onion` check so that a clearnet server not yet geolocated is not
    silently misclassified as tor. (See AOQ-2.)

### Persistence & URL

- **FR-9** Custom filters SHALL be persisted in `localStorage`, per-browser,
  not synced across devices or shared. (Traces: Q6=A.) The stored payload MUST
  survive reload and be independent of the existing `serversFilter`
  "last used" key so that saved filters and the last-used filter do not
  clobber each other.
- **FR-10** Applying any preset, custom filter, or ad-hoc filter SHALL update
  the page URL query params exactly as the current filter does today, so every
  resulting view is linkable and bookmarkable; opening such a URL SHALL restore
  that view. (Traces: Q7=A.) The URL remains the source of truth for the
  *active* filter; custom-filter *definitions* live in `localStorage` (FR-9).
- **FR-11** The existing default view (`status = up`) SHALL be preserved when
  no filter is supplied by URL, and the feature SHALL be purely additive to the
  current filter mechanics — no existing filter capability is removed.

### Analytics

- **FR-12** On every preset apply and every custom-filter apply, the system
  SHALL emit a lightweight analytics event via the existing
  Google-Analytics-style measurement id (`VITE_ANALYTICS_MEASHUREMENT_ID`),
  carrying an identifier of which preset or custom filter was applied, so
  post-release usage can be measured. (Traces: Q8=A, and the intent's
  metrics goal.) When the measurement id is not configured, emission SHALL be a
  safe no-op (no error surfaced to the user).

## Non-Functional Requirements

- **NFR-1 (Layering)** Filter-preset and custom-filter logic (definitions,
  persistence, the derived `network` classification, apply/serialize) SHALL
  live in the store layer under `src/store/servers/` (service + state), not in
  component-local logic; presentation lives under `src/components/servers/`.
  Database `snake_case` field names MUST NOT leak past the service boundary.
  (Traces: team Code Style practices.)
- **NFR-2 (Testing)** All new/changed logic SHALL be developed test-first
  (TDD) with Vitest; UI with `@testing-library/svelte`. New/changed code SHALL
  meet the 80% line-coverage floor. CI MUST fail the PR if tests or
  `svelte-check` do not pass.
- **NFR-3 (No backend change)** The feature SHALL NOT require any Supabase
  schema, view, or query change; all preset/custom-filter behaviour is achieved
  with the existing data and client-side derivation.
- **NFR-4 (Accessibility)** Preset and custom-filter controls SHALL be
  keyboard-operable and expose their selected/active state to assistive
  technology (consistent with UIkit component usage already in the app).
- **NFR-5 (Graceful storage failure)** If `localStorage` is unavailable or
  contains malformed data, the page SHALL still render and function with
  presets only; custom-filter reads/writes MUST fail safe without breaking the
  servers list.

## Assumptions & Open Questions

- **AOQ-1** "Recently Added" is specified as a sort (newest first) rather than a
  filter, because no `createdAt` filter field exists in the `Filter` model and
  the intent did not call for a fixed recency cutoff. Assumption: a sort-based
  view satisfies the user's "recently added" need. To confirm at mockups/design.
- **AOQ-2** Tor/clearnet is derived from location presence per the user's Q3
  direction; the host-`.onion` suffix is more authoritative. Assumption: design
  will combine them (host-`.onion` ⇒ tor; otherwise location presence decides)
  so un-geolocated clearnet servers are not misclassified. To resolve in
  domain/functional design.
- **AOQ-3** Custom-filter names are assumed free-text, non-unique-enforced but
  practically distinguishable; whether duplicate names are blocked is a design
  detail deferred to mockups.

## Traceability

| Requirement | Origin |
|---|---|
| FR-1, FR-2, FR-5 | Intent ("instantly apply a predefined filter"); Q1 |
| FR-3 | Q2 (five named presets) |
| FR-6, FR-7 | Intent ("create, save, edit, and persist custom filters"); Q4, Q5 |
| FR-8 | Q3 (location-derived tor/clearnet) |
| FR-9 | Intent ("persist … in localStorage"); Q6 |
| FR-10 | Existing URL-encoded filter behaviour; Q7 |
| FR-12 | Intent (measure usage); Q8 |
| NFR-1..NFR-5 | Team practices (layering, TDD/coverage, no-backend-change scope) |

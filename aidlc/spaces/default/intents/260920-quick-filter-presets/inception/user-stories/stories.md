# User Stories — Servers Quick-Filter Presets & Custom Filters

## Sources

- Requirements `../requirements-analysis/requirements.md` (FR/NFR IDs referenced per story)
- Personas `personas.md` (P1 Catalog Visitor)
- Plan answers `user-stories-questions.md`
- Team practices `../practices-discovery/team-practices.md` (TDD/testability posture and layering conventions shaping the acceptance criteria)
- Mob contributions `contributions/aidlc-design-agent.md`, `contributions/aidlc-developer-agent.md`, `contributions/aidlc-quality-agent.md` (folded in below)

## Assumptions & Open Questions

None. (Two design-level items — duplicate custom-filter names, and the exact
overlay-marker set for clearnet — are recorded in the requirements as AOQ-3 and
AOQ-4 and are settled or owned by Refined Mockups; they are not story-level open
questions.)

## Conventions

- All stories share the single persona **P1 Catalog Visitor**.
- Priority is MoSCoW; Delivery Planning is skipped in this scope, so priority is
  the de-facto build order.
- Acceptance criteria use Given/When/Then. IDs: stories `US{group}.{seq}`,
  criteria `AC{group}.{seq}.{n}`.
- "View" = the active filter **and** sort together (what the URL encodes).

---

## Group 1 — Presets  (FR1, FR2, FR3, FR4)

### US1.1 — Apply a preset in one click  · Must Have
As a catalog visitor, I want to click a quick-filter preset button, so that the
server list instantly shows that predefined view without my building a filter by hand.

- **AC1.1.1** — Given the servers page is loaded, When I look at the filter area, Then a row of preset buttons is visible: All Online, Online Clearnet, Online Tor, Recently Added, High Uptime.
- **AC1.1.2** — Given any current view, When I click a preset, Then the list updates immediately (no confirm step) to that preset's filter and sort.
- **AC1.1.3** — Given I clicked a preset, When the view updates, Then the entire previous filter and sort are replaced by the preset's (not merged with any prior facet).
- **AC1.1.4** — Given a preset is active, When I click a different preset, Then the first preset's view is fully cleared and replaced by the second (presets are mutually exclusive).
- **AC1.1.5** — Given a preset whose criteria match no servers, When I click it, Then the list shows an empty result (no error), and the preset is still indicated as the active/selected view.
- **AC1.1.6** — Given I apply a preset or custom filter, When the view updates, Then the filter and the sort are applied together as one coherent view transition (the resulting list and URL reflect both at once; there is no intermediate state where only one of filter/sort has changed).

### US1.2 — Preset definitions match the intended slices  · Must Have
As a catalog visitor, I want each preset to show exactly the slice its name
implies, so that I can trust one click to give the right servers.

- **AC1.2.1** — Given I click **All Online**, Then results are servers with status = online, sorted by last check (newest first).
- **AC1.2.2** — Given I click **Online Clearnet**, Then results are online servers whose location is not an overlay-network marker (location is not TOR, I2P, or YGGDRASIL), sorted by last check. Note: this is the existing `countries` exclusive filter, so a server with an empty/unknown location is treated as clearnet, consistent with the confirmed rule "location is not Tor/I2P" (requirements AOQ-4).
- **AC1.2.3** — Given I click **Online Tor**, Then results are online servers whose location is TOR, sorted by last check.
- **AC1.2.4** — Given I click **Recently Added**, Then results are online servers sorted by creation date, newest first.
- **AC1.2.5** — Given I click **High Uptime**, Then results are online servers with 90-day uptime ≥ 90% (90.0 included at the boundary), sorted by 90-day uptime, highest first. (Threshold confirmed per requirements AOQ-5.)
- **AC1.2.6** — Given any preset is applied, Then it is expressed using the existing filter and sort model only (no backend/query change is required for it to work).

---

## Group 2 — Custom-filter lifecycle  (FR6, FR7)

### US2.1 — Save the current view as a named custom filter  · Must Have
As a catalog visitor, I want to save my current filter and sort under a name, so
that I can re-apply my own recurring view later with one click.

- **AC2.1.1** — Given I have configured any filter/sort via the normal controls, When I choose "save current filter" and enter a name, Then a new custom-filter button with that name appears alongside the presets.
- **AC2.1.2** — Given I saved a custom filter, When I reload the page, Then the custom-filter button is still present (persistence, see US3.1).
- **AC2.1.3** — Given I save a custom filter, Then it captures both the filter and the current sort (a full view), consistent with presets.
- **AC2.1.4** — Given I save a custom filter, Then it is assigned a stable internal id at creation time that does not change when the filter is later renamed or edited (this id is what analytics AC4.1.2 and rename/edit US2.4/US2.5 key on).
- **AC2.1.5** — Given the save dialog, When I try to save with an empty or whitespace-only name, Then the save is rejected and no custom filter is created.
- **AC2.1.6** — Given the servers page (including the first visit with zero saved custom filters), Then the "save current filter" affordance is discoverable in the filter area, so a first-time or occasional visitor can find the save capability.

### US2.2 — Apply a custom filter  · Must Have
As a catalog visitor, I want to click a saved custom filter, so that its view is
applied instantly.

- **AC2.2.1** — Given a saved custom filter exists, When I click it, Then its view (filter + sort) replaces the entire current view, exactly like a preset (US1.1).
- **AC2.2.2** — Given I saved a custom filter, closed the page, and reopened it in the same browser, When I click that custom filter, Then it applies the exact view (filter + sort) that was saved (round-trip through storage).

### US2.3 — Delete a custom filter  · Must Have
As a catalog visitor, I want to delete a custom filter I no longer need, so that
my button row stays relevant.

- **AC2.3.1** — Given a saved custom filter, When I delete it, Then its button disappears and it does not return after reload.
- **AC2.3.2** — Given I delete the custom filter whose view is currently active, When it is removed, Then the currently displayed result set and the URL are identical to just before the delete (only the saved definition is removed, not the active view).
- **AC2.3.3** — Given a custom-filter button, Then its delete (and rename/edit) control can be invoked without that invocation also applying the filter's view (managing a custom filter is distinct from applying it).
- **AC2.3.4** — Given I invoke delete on a custom filter, Then the removal requires an explicit confirmation step before the filter is deleted (delete is irreversible; an accidental click does not lose a saved filter).

### US2.4 — Rename a custom filter  · Should Have
As a catalog visitor, I want to rename a custom filter, so that its label keeps
matching how I think about it.

- **AC2.4.1** — Given a saved custom filter, When I rename it, Then the button shows the new name and the saved definition (filter + sort) and its stable id are unchanged.
- **AC2.4.2** — Given I rename a custom filter, When I reload, Then the new name persists.
- **AC2.4.3** — Given the rename input, When I submit an empty or whitespace-only name, Then the rename is rejected and the previous name is kept.

### US2.5 — Edit a custom filter in place  · Should Have
As a catalog visitor, I want to re-save my current view over an existing custom
filter, so that I can update it without deleting and recreating it.

- **AC2.5.1** — Given a saved custom filter and a different current view, When I re-save over that custom filter, Then its stored definition is replaced by the current view and its name and stable id are kept.
- **AC2.5.2** — Given I edited a custom filter in place, When I later click it, Then it applies the updated view.

---

## Group 3 — Persistence & URL round-trip  (FR9, FR10, FR11, NFR5)

### US3.1 — Custom filters persist per browser  · Must Have
As a catalog visitor, I want my custom filters stored in my browser, so that they
survive reloads on this device.

- **AC3.1.1** — Given I have saved custom filters, When I close and reopen the page in the same browser, Then all my custom filters are still listed.
- **AC3.1.2** — Given custom filters are stored, Then they are kept independently of the existing "last used filter" storage (a separate storage key) so the two never overwrite each other.
- **AC3.1.3** — Given browser storage is unavailable or the stored custom-filter data is malformed, When the servers page loads, Then the page still renders and functions with presets only, custom-filter reads/writes fail safe, and no error is surfaced to the user or breaks the servers list (NFR5).

### US3.2 — Every view is linkable via URL  · Must Have
As a catalog visitor, I want the URL to reflect whatever view is active, so that I
can bookmark or share a specific view.

- **AC3.2.1** — Given I apply a preset, a custom filter, or an ad-hoc filter, When the view changes, Then the URL updates to encode that view (filter params + sort params), exactly as manual filter/sort changes do today.
- **AC3.2.2** — Given a URL that encodes a view, When I open it, Then the servers list shows that exact view (filter + sort).
- **AC3.2.3** — Given I apply a preset or custom filter, Then the URL contains only the resulting filter/sort — never the preset's or custom filter's name or id.
- **AC3.2.4** — Given two presets that differ only by sort (e.g. All Online vs Recently Added), When each is applied, Then their URLs differ (sort is part of the URL) so neither collides with the other.

### US3.3 — Default view preserved  · Must Have
As a catalog visitor, I want the page to keep its current default behaviour when I
arrive with no view specified, so that nothing regresses.

- **AC3.3.1** — Given I open the servers page with no view in the URL, Then the existing default view (status = online, sorted by last check) is shown.
- **AC3.3.2** — Given the feature is added, Then all existing filter and sort controls continue to work unchanged (the feature is additive).

---

## Group 4 — Analytics  (FR12)

### US4.1 — Measure which filters get used  · Should Have
As the product owner (via the catalog visitor's actions), I want an analytics
event on each preset/custom-filter apply, so that after release I can see which
filters are used most.

- **AC4.1.1** — Given analytics is configured, When a preset is applied, Then an event is emitted via the existing Google-Analytics-style measurement id, identifying which preset.
- **AC4.1.2** — Given analytics is configured, When a custom filter is applied, Then an event is emitted carrying the custom filter's stable internal id (AC2.1.4), not the user's editable name; the display name may be included as a separate property.
- **AC4.1.3** — Given the analytics measurement id is not configured, When a filter is applied, Then no error is surfaced to the user (safe no-op).
- **AC4.1.4** — Given any apply event is emitted, Then the applied filter identifier is carried in the event only and never written to the URL.
- **AC4.1.5** — Given I change the view via the normal filter/sort controls (an ad-hoc change, not a preset or custom-filter click), Then no preset/custom-filter apply event is emitted (only preset and custom-filter applies are counted).

---

## Group 5 — Selected-state feedback  (FR5, FR5.1, NFR4)

### US5.1 — See which preset/custom filter is active  · Must Have
As a catalog visitor, I want the currently active preset or custom filter shown as
selected, so that I know which view I am looking at.

- **AC5.1.1** — Given the active view matches a preset's definition, Then that preset button is shown as selected.
- **AC5.1.2** — Given the active view matches a saved custom filter's definition, Then that custom-filter button is shown as selected.
- **AC5.1.3** — Given the active view matches no preset and no custom filter (e.g. I hand-edited it), Then no button is shown as selected.
- **AC5.1.4** — Given the active view matches both a preset and a custom filter, Then only the preset is shown as selected (preset precedence); at most one button is selected at a time.
- **AC5.1.5** — Given selected-state matching, Then a view matches a definition only when BOTH its filter and its sort match, and the match is insensitive to the order of values within a multi-value filter facet.
- **AC5.1.6** — Given I open the servers page with the default view (no view in the URL), Then, because the default view equals the All Online preset, the All Online preset is shown as selected on load.
- **AC5.1.7** — Given the preset and custom-filter buttons, Then each is keyboard-operable (focusable and activatable via the keyboard) (NFR4).
- **AC5.1.8** — Given the custom-filter management controls (save affordance, name input, rename, edit, delete), Then each is keyboard-operable and labelled for assistive technology (NFR4).
- **AC5.1.9** — Given the preset and custom-filter buttons, Then each exposes its selected/active state to assistive technology (e.g. via an appropriate pressed/selected ARIA state) (NFR4).

---

## INVEST notes

- **Independent**: Group 1 (presets) ships without Groups 2/4. Group 5 (selected
  state) depends on the notion of an active view but not on custom filters
  existing. Group 4 (analytics) hooks the apply action shared by Groups 1–2.
- **Negotiable / Valuable / Estimable / Small**: each story is a single
  user-visible capability with concrete ACs; none bundles unrelated behaviour.
- **Testable**: every AC is a Given/When/Then with an observable outcome,
  supporting the TDD posture.

## Dependencies

- US2.2/US2.3/US2.4/US2.5 depend on US2.1 (a custom filter must exist first; the
  stable id from AC2.1.4 underpins US2.4/US2.5 and AC4.1.2).
- US5.1 depends on US1.1 (presets) and US2.2 (custom apply) for the match set.
- US3.1 underpins US2.1 persistence; US3.2 applies to every apply (US1.1, US2.2).
- US4.1 hooks the apply action of US1.1 and US2.2.

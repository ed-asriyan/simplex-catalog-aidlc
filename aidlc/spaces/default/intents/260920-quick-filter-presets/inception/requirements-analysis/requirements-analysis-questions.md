# Requirements Analysis — Questions

## Sources

- Intent statement: `ideation/intent-capture/intent-statement.md`
- Team practices: `inception/practices-discovery/team-practices.md`
- Current filter model inspected in `simplex-catalog-frontend/src/store/servers/servers-service.ts` (the `Filter` interface) and `src/components/servers/list.svelte` (how the filter is applied, URL-synced, and persisted).

**Context on what the servers filter can express today** (the `Filter` interface): `labels` (include/exclude a set), `status` (up / down / unknown), `countries` (include/exclude a set), `identity` (substring), `host` (substring), `uuid`, `infoPageAvailable` (yes/no), `protocol` (`smp` / `xftp`), and minimum uptime over 7 / 30 / 90 days (`uptime7/30/90` as a "≥ N%" threshold). The current filter is stored in the page URL and mirrored to one `localStorage` key (`serversFilter`) as the "last used" filter; the default when nothing is set is "status = up".

There is **no explicit "tor vs clearnet" field**. Tor servers use `.onion` host addresses; clearnet servers do not. The host filter is a substring match with no "does NOT contain" option, so "only tor" is expressible (host contains `.onion`) but "clearnet only" is not directly expressible with today's model. Q3 asks how to handle this.

## Q1. Interaction model — how should clicking a quick-filter preset behave relative to the current filter?

Jira's Quick Filters toggle/combine; your description says a preset "immediately applies that preset's view on click."

- A. Clicking a preset REPLACES the entire current filter with the preset's filter (one click = that exact view), matching "instantly apply a predefined filter" — presets are mutually exclusive, clicking one clears the others
- B. Clicking a preset TOGGLES it and multiple presets combine (AND) with each other and with manual filters, like Jira
- C. Something else
- X. Other (please specify)

[Answer]: A. Clicking a preset REPLACES the entire current filter. Presets are mutually exclusive: applying one clears the others and any manual filter, giving exactly that preset's view in one click.

## Q2. Which hardcoded presets should ship? (select all that apply)

Based on the filter fields that actually exist, here's a proposed starter set. Pick the ones you want; you can also add/rename in "Other".

- A. **Online** — status = up (this is close to today's default view)
- B. **High uptime (90d ≥ 90%)** — uptime90 ≥ 90
- C. **Tor only** — host contains `.onion`
- D. **SMP servers** — protocol = smp; and **XFTP servers** — protocol = xftp (two protocol presets)
- E. **Has info page** — infoPageAvailable = yes
- X. Other (please specify / rename / add presets, e.g. "Online clearnet", specific countries)

[Answer]: The shipped preset set is five. Each preset is a full VIEW = filter + sort, and every preset filters to online servers (status = true):
1. **All Online** — status = online; sort by last_check desc.
2. **Online Clearnet** — status = online AND location is clearnet (location not TOR/I2P/YGGDRASIL); sort by last_check desc.
3. **Online Tor** — status = online AND location = TOR; sort by last_check desc.
4. **Recently Added** — status = online; sort by created_at desc (no recency-window filter — it is a sort).
5. **High Uptime** — status = online AND uptime90 ≥ 90; sort by uptime90 desc.
The earlier SMP/XFTP and "Has info page" suggestions are NOT shipped as presets. Sorting is explicitly part of each preset (refinement). The tor/clearnet distinction uses the location field directly — see Q3.

## Q3. How should the "clearnet" (non-tor) case be handled, given the filter can't currently express "host does NOT contain .onion"?

Your dogfooding examples included "all active clearnet."

- A. Skip a clearnet preset for now — ship "Tor only" but no "clearnet only" (keeps this feature purely additive, no filter-model change)
- B. Extend the filter model to support tor/clearnet as a first-class field (adds backend/query work — larger scope)
- C. Approximate clearnet another way — I'll explain
- X. Other (please specify)

[Answer]: Use the location field directly — this is more precise and technically correct, and needs no new "network" term. The location (`country`) field already carries overlay-network marker strings — `TOR`, `I2P`, `YGGDRASIL` (see utils.ts getFlagEmoji) — instead of a geographic code for servers reachable only over those networks. So:
- **clearnet** = location is a real country code, i.e. location is NOT TOR/I2P/YGGDRASIL.
- **tor** = location = TOR.
Both are expressible TODAY with the existing inclusive/exclusive `countries` filter (Online Clearnet = countries exclusive of [TOR, I2P, YGGDRASIL]; Online Tor = countries inclusive [TOR]). No new filter field, no `.onion` heuristic, no backend change. Because the location marker is authoritative, there is no misclassification of un-geolocated servers. This removes the earlier proposed derived `network` requirement (FR-8 withdrawn).

## Q4. Custom filters — what does "create/save a custom filter" capture, and how is it created?

- A. The user configures the normal filters (status, country, uptime, etc.), clicks "Save current filter", names it, and it's stored as a reusable custom quick-filter button alongside the presets
- B. A dedicated form/dialog to build a filter from scratch (separate from the main filter controls), then save it
- C. Something else
- X. Other (please specify)

[Answer]: A. The user configures the normal filter controls, clicks "Save current filter", gives it a name, and it appears as a reusable custom quick-filter button alongside the hardcoded presets.

## Q5. Custom filter management — which operations, and any limits?

- A. Create, rename, edit (re-save over an existing one), and delete; no hard limit on count
- B. Same operations, but cap the number of saved custom filters (I'll say how many)
- C. Create and delete only (no in-place edit — to change one, delete and re-create)
- X. Other (please specify)

[Answer]: A. Full CRUD — create, rename, edit-in-place (re-save over an existing one), and delete — with no hard limit on how many custom filters a user can save.

## Q6. Persistence scope — you confirmed custom filters live in localStorage. Anything about that to pin down?

- A. localStorage only, per-browser, not synced across devices or shared — that's expected and fine
- B. localStorage now, but design it so a future server-side sync could be added without breaking saved filters
- X. Other (please specify)

[Answer]: A. Plain localStorage only — per-browser, not synced across devices or shared. That is the expected and accepted behaviour; no server-side sync is planned for this intent.

## Q7. Do the ad-hoc (unsaved) filters and the presets/custom filters need to stay shareable via URL, as the current filter is?

Today the active filter is encoded in the URL so a filtered view can be linked/bookmarked.

- A. Yes — applying any preset or custom/ad-hoc filter should update the URL the same way, so any view stays linkable/bookmarkable
- B. No — presets/custom filters are a local convenience only; the URL doesn't need to reflect them
- X. Other (please specify)

[Answer]: A. Yes — applying any preset, custom filter, or ad-hoc unsaved filter updates the URL exactly as the current view does today, so every resulting view stays linkable and bookmarkable. REFINEMENT: the URL encodes ONLY the resulting view — the filter query params plus the existing sortField/sortOrder params (sort is already URL-synced). The preset/custom-filter name or id is NEVER written to the URL; a button is just a shortcut that sets the underlying filter+sort, and does not change how state integrates with the URL.

## Q8. Analytics — you want to measure which filters/presets get used most after release. How should that be captured?

The frontend already ships Sentry (`@sentry/svelte`) and references a `VITE_ANALYTICS_MEASHUREMENT_ID` (a Google-Analytics-style measurement id) in settings.

- A. Emit a lightweight analytics event on preset/custom-filter apply via the existing analytics measurement id (Google Analytics-style), naming which preset was applied
- B. Use Sentry for it
- C. Defer analytics wiring to a later intent — just make sure preset applies are structured so an event can be added later without rework
- X. Other (please specify)

[Answer]: A. Emit a lightweight analytics event on every preset / custom-filter apply via the existing Google-Analytics-style measurement id (`VITE_ANALYTICS_MEASHUREMENT_ID`), carrying which preset or custom filter was applied so post-release usage ("which filters are used most") can be measured.

## Consolidated Summary Confirmation

The eight answers above resolve to this design: preset click REPLACES the whole VIEW — filter AND sort (presets mutually exclusive). Five shipped presets, each a filter+sort, all filtered to status=online: All Online (sort last_check), Online Clearnet (location not TOR/I2P/YGGDRASIL; sort last_check), Online Tor (location=TOR; sort last_check), Recently Added (sort created_at), High Uptime (uptime90≥90; sort uptime90). Tor/clearnet comes straight from the location field's overlay markers via the existing inclusive/exclusive `countries` filter — no `network` term (FR-8 withdrawn), no `.onion` heuristic, no backend change. Custom filters: "save current filter" + name, capturing filter+sort; full CRUD, no count limit; plain per-browser localStorage. URL encodes the resulting VIEW ONLY (filter params + existing sortField/sortOrder) — never a preset/custom name; a button is just a shortcut. GA-style analytics event on every apply. Two flagged confirmations: AOQ-4 (clearnet excludes YGGDRASIL too) and AOQ-5 (High Uptime keeps the ≥90 filter, not sort-only).

[Answer]:

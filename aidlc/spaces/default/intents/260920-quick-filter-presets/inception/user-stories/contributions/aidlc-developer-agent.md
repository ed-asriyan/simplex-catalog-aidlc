**Collaborator:** aidlc-developer-agent

## Contribution

Reviewed for implementability against the existing `Filter`/`Sort`/`Server`
model (`servers-service.ts`) and the filter/sort/URL/localStorage wiring
(`list.svelte`). The draft is implementable end-to-end with no backend or model
change (confirms AC1.2.6, FR-4, NFR-3). The five preset definitions map cleanly
onto the existing model: `status`, the inclusive/exclusive `countries`
`FilterArray` (client-side filter, `servers-service.ts:194-197`), the
server-side `uptime90` `gte` (`:128-130`), and the `last_check`/`created_at`/
`uptime90` sort fields. Sizing is good: the CRUD split (US2.1–US2.5) and the
presets/definitions split (US1.1/US1.2) are each an independently testable,
appropriately small capability. Concrete gaps and notes below, at AC altitude.

**1. US2.1 must assign a stable custom-filter id (data-model gap).** AC4.1.2
requires the apply event to carry "a stable internal id for that custom filter
(not the user's editable name)", and US2.4/US2.5 require rename/edit to preserve
identity — but no story establishes that a stable id is *minted at creation* and
kept across rename and edit-in-place. Suggest adding an AC to US2.1, e.g.
**AC2.1.4** — "When a custom filter is created, it is assigned a stable internal
id that is independent of its display name and is preserved across rename
(US2.4) and edit-in-place (US2.5)." Without it, AC4.1.2's id has no origin.

**2. Applying a preset/custom filter writes filter and sort through two
separate code paths today.** In `list.svelte`, `updateFilter()` writes the
filter query params *plus* `localStorage['serversFilter']`, while `updateSort()`
writes only `sortField`/`sortOrder`. A preset/custom apply is a single
"replace the whole view" action (US1.1/US2.2, FR-2) that must set filter **and**
sort together. This is implementable but is not a single existing call; worth an
AC clarifying that one apply produces one coherent view update (filter + sort in
the same URL transition), so it is not read as two independent user actions.

**3. Selected-state matching (US5.1) must key on filter AND sort, and be
value-order-insensitive.** All Online and Recently Added share the same filter
and differ only by sort (AC1.2.1 vs AC1.2.4); the matcher in AC5.1.1 must
therefore compare sort too, or both would light up. AC5.1 currently says "the
active view matches a preset's definition" and relies on the Conventions
definition of "view" = filter + sort — recommend making "filter **and** sort"
explicit in AC5.1.1/AC5.1.2 so the TDD spec is unambiguous. Also, `countries`/
`labels` are `FilterArray.values` string arrays that round-trip via
`.split(',')`; a custom filter built from ad-hoc controls may hold the same
values in a different order than a preset, so the match must canonicalize
(order-insensitive on `values`) to avoid false negatives. (The two `Symbol()`
keys injected into the derived `filter` in `list.svelte:60-64` are dropped by
`JSON.stringify`/`Object.keys`, so they do not interfere with equality.)

**4. Default landing view is identical to the All Online preset.** The default
(`status = true`, `last_check` desc — AC3.3.1) is byte-for-byte the All Online
preset (AC1.2.1). Per AC5.1.1, a visitor arriving with no URL would see **All
Online shown as selected** on first load. That is arguably desirable, but it is
an unstated consequence — recommend an explicit AC on US5.1 (or a note on
US3.3) stating that the default view resolves to the All Online preset selected.

**5. "Clearnet" via the exclusive `countries` filter also matches
empty/unknown-location servers.** `server.country` defaults to `''` when a
status has no country (`servers-service.ts:176`), and the clearnet filter is
`!['TOR','I2P','YGGDRASIL'].includes(country)`, so an online server with an
empty/unknown location is **included** in Online Clearnet. AC1.2.2's operative
parenthetical ("location is not TOR, I2P, or YGGDRASIL") is exactly what the
code does, but the surrounding prose "location is a real country" is stronger
than the implementation. Recommend aligning the prose to "not an overlay
marker" (matches AOQ-2/AOQ-4), or, if empty-location servers should be excluded,
saying so — it changes the filter expression.

**6. US3.1 has no AC for the malformed/unavailable-storage path.** NFR-5
mandates fail-safe behaviour, and this is a Must-Have persistence story, but
US3.1's ACs only cover the happy path. Recommend an edge AC, e.g. **AC3.1.3** —
"Given custom-filter storage is unavailable or malformed, When the page loads,
Then presets still render and function and the servers list is unaffected (no
error surfaced)" — so the fail-safe is directly TDD-testable and doesn't rely on
NFR prose alone. This also gives the store-layer read a defined contract for the
"plain storage, no versioning" decision (a malformed/legacy blob is discarded,
not migrated).

**7. Minor — High Uptime null handling is already correct.** `uptime90` may be
`null` (no check in the 90-day window; `servers-service.ts:57`). The server-side
`gte('uptime90', 90)` excludes nulls and the `uptime90` desc sort sinks any
residual nulls to the bottom, so AC1.2.5 needs no change; noting it so it is not
raised later as an edge miss.

None of the above blocks the draft; items 1, 3, and 6 are the ones I would most
want reflected in ACs before Code Generation, since each pins a data-model or
edge-case contract the TDD suite will need.

## Positions

AGREE: The draft is implementable as written against the existing model with no backend change; the observations above are AC-completeness refinements, not corrections to its direction.

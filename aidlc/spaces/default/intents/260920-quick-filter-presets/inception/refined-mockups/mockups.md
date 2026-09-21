# Refined Mockups — Servers Quick-Filter Presets & Custom Filters

Mid-fidelity textual wireframes for the servers page. The app is Svelte 5 +
UIkit; these mockups describe layout, states, and copy — the concrete UIkit
components are in `design-system-mapping.md`, the behaviour in
`interaction-spec.md`, and a11y in `accessibility-checklist.md`.

## Sources

- User stories `../user-stories/stories.md` (US1.1–US5.1; AC ids referenced inline)
- Requirements `../requirements-analysis/requirements.md` (FR1–FR12, NFR4)
- Team practices `../practices-discovery/team-practices.md` (UIkit, kebab-case, feature layering)
- Existing UI: `src/components/servers/table/index.svelte` (filter card + toolbar),
  `src/components/servers/list.svelte` (filter/sort/URL wiring)

Note: rough-mockups/wireframes/user-flow are absent by scope design; these are
designed from the stories + requirements + the existing UI.

## Assumptions & Open Questions

None. (Duplicate custom-filter names → requirements AOQ-3, deferred to
implementation; clearnet overlay set → AOQ-4, settled.)

## Layout overview

The quick-filter bar is inserted ABOVE the existing filter card (Q1=A). Nothing
below it changes structurally; applying a chip drives the existing filter card
and sort state.

```
Servers Catalog  🌐
Discover and share community-run SMP and XFTP servers...
[ Add server anonymously ]

┌─ Quick filters ────────────────────────────────────────────────────────────┐
│  [All Online] [Online Clearnet] [Online Tor] [Recently Added] [High Uptime]  │  ← presets (US1.1)
│  · [My clearnet 90+ ⋯] [Tor watchlist ⋯]              [ + Save current ]      │  ← custom chips (US2.1) + save
└──────────────────────────────────────────────────────────────────────────────┘
┌─ Filters (existing card) ────────────────────────────────────────────────────┐
│ URI[____] Identity[____] Type[▾] Status[▾] Info[▾] Location[▾] Labels[▾]      │
│ Min uptime 7d[__] 30d[__] 90d[__]                                             │
└──────────────────────────────────────────────────────────────────────────────┘
[🔄] [Import labels] [Export labels]                 [Page size ▾] [Bulk mode]
┌─ Table ───────────────────────────────────────────────────────────────────────┐
│ ▢ Status  Host  Identity  Country  Type  Uptime7/30/90  Info  Added  …         │
│ … rows …                                                                       │
└──────────────────────────────────────────────────────────────────────────────┘
```

## M1 — Quick-filter bar, default load  (US1.1, US5.1, AC5.1.6)

On first load with no URL view, the default view equals **All Online**, so that
preset renders selected (primary style, `aria-pressed="true"`).

```
Quick filters
[✔ All Online] [Online Clearnet] [Online Tor] [Recently Added] [High Uptime]
                                                              [ + Save current ]
```
- `[✔ All Online]` = active chip (uk-button-primary, aria-pressed=true).
- Others = uk-button-default, aria-pressed=false.
- No custom chips yet → row shows presets + the "+ Save current" affordance only
  (empty-state, AC2.1.6). The "+ Save current" control is always visible.

## M2 — A preset applied  (US1.1, US1.2, AC1.1.2–AC1.1.4)

Clicking **Online Tor**: the filter card's Status→Active and Location→(TOR,
inclusive) update, sort stays last_check desc, table reloads, URL updates.

```
[All Online] [Online Clearnet] [✔ Online Tor] [Recently Added] [High Uptime]
Filters:  Status[Active ▾]   Location[1 selected ▾ = TOR]   …
Total servers matching filters: 42
```

## M3 — Empty result for a preset  (AC1.1.5)

```
[All Online] [Online Clearnet] [✔ Online Tor] [Recently Added] [High Uptime]
Table: (no rows)
Total servers matching filters: 0
```
- Chip stays selected; table area shows the existing "Total … : 0" affordance,
  no error.

## M4 — Save current filter  (US2.1, AC2.1.1, AC2.1.5)

User has an ad-hoc view (e.g. Status=Active, Min uptime 90d=95). Clicks
**+ Save current** → native `prompt("Name this filter")`.

```
┌ prompt ─────────────────────────────┐
│ Name this filter:                    │
│ [ My clearnet 90+_______________ ]   │
│                    [Cancel] [ OK ]   │
└──────────────────────────────────────┘
```
- On OK with a non-empty name → a new chip appears; it becomes the selected chip
  (its definition now equals the active view).
- Empty/whitespace name → save rejected, no chip created (AC2.1.5); re-prompt or
  no-op (native prompt returns empty/null → treated as cancel).

## M5 — Custom chip with ⋯ menu  (US2.2–US2.5, AC2.3.3)

```
[ My clearnet 90+ ⋯ ]
        └─▾ dropdown ─────────────┐
          │ Rename…               │  → prompt() (US2.4)
          │ Update to current view│  → confirm/apply (US2.5)
          │ Delete                │  → confirm() (US2.3, AC2.3.4)
          └───────────────────────┘
```
- Clicking the chip **body** applies the saved view (US2.2).
- Clicking **⋯** opens the menu WITHOUT applying the filter (AC2.3.3).

## M6 — Delete confirmation  (US2.3, AC2.3.4)

```
┌ confirm ────────────────────────────────────────┐
│ Delete filter "My clearnet 90+"? This cannot be  │
│ undone.                          [Cancel] [ OK ] │
└──────────────────────────────────────────────────┘
```
- OK → chip removed, persisted removal (AC2.3.1). If it was the active view, the
  displayed results and URL are unchanged (AC2.3.2); the chip simply deselects.

## M7 — Modified view after applying a preset or custom filter  (US1.3, US2.6)

User applied **Online Tor**, then changes Min uptime 90d in the filter card.

```
[All Online] [Online Clearnet] [Online Tor] [Recently Added] [High Uptime]
   (nothing selected — active view no longer matches any chip; AC1.3.2/AC2.6.2)
Filters:  Status[Active] Location[TOR] Min uptime 90d[90]   ← ad-hoc
```
- No chip is highlighted (unless the edited view coincidentally equals another
  chip's definition).
- For a **custom** filter modified this way, its saved definition is untouched;
  the ⋯ menu's "Update to current view" (US2.5) or "+ Save current" (US2.1) are
  the ways to persist; re-clicking the chip restores its saved view (AC2.6.4).
- URL reflects the ad-hoc view (AC1.3.4/AC2.6.5). No analytics apply event fires
  for the ad-hoc edit (AC4.1.5).

## M8 — Storage unavailable / malformed  (US3.1, AC3.1.3, NFR5)

```
Quick filters
[✔ All Online] [Online Clearnet] [Online Tor] [Recently Added] [High Uptime]
                                                              [ + Save current ]
```
- Presets always render. If localStorage is unavailable or the stored custom
  filters are malformed, custom chips are simply absent (no error toast, page
  fully functional). Saving may be a no-op if storage cannot be written; it must
  not throw.

## M9 — Rename a custom filter  (US2.4, AC2.4.1, AC2.4.3)

From the ⋯ menu → Rename… → native `prompt` pre-filled with the current name.

```
┌ prompt ─────────────────────────────┐
│ Rename filter:                       │
│ [ My clearnet 90+_______________ ]   │  ← pre-filled with current name
│                    [Cancel] [ OK ]   │
└──────────────────────────────────────┘
```
- OK with non-empty name → chip label updates; saved definition + stable id
  unchanged (AC2.4.1). Empty/whitespace → rejected, previous name kept (AC2.4.3).

## M10 — Update a custom filter to the current view  (US2.5, AC2.5.1)

User applied "My clearnet 90+", tweaked the filter card, then ⋯ → Update to
current view.

```
[ My clearnet 90+ ⋯ ]
        └─▾ [ Rename… ] [ Update to current view ] [ Delete ]
                              └── replaces saved definition with current
                                  {filter, sort}; name + id kept (AC2.5.1)
```
- After update, clicking the chip applies the new view (AC2.5.2); the chip
  becomes selected again (its definition now equals the active view).

## M11 — Selected-state precedence when a view matches both  (US5.1, AC5.1.4)

If the active view happens to equal BOTH a preset and a saved custom filter, only
the preset is shown selected.

```
active view == All Online's definition == custom "Everything" definition
[✔ All Online] [Online Clearnet] … · [ Everything ⋯ ]   ← custom NOT highlighted
        └ selected (preset precedence, aria-pressed=true)   └ aria-pressed=false
```
- At most one chip is `aria-pressed="true"` at any time (AC5.1.4).

## Responsive behaviour

- The quick-filter bar wraps (`uk-flex` + `uk-flex-wrap`, `gap`) like the
  existing toolbar; on narrow (phone) widths chips wrap to multiple lines, the
  "+ Save current" control stays at the end of the row.
- No horizontal scroll introduced; chips are tap-targets ≥ 32px tall.
- The existing filter card already uses responsive `uk-child-width-*@s/@m/@l`;
  unchanged.

## Copy

- Preset labels: `All Online`, `Online Clearnet`, `Online Tor`,
  `Recently Added`, `High Uptime`.
- Save control: `+ Save current`. Menu items: `Rename…`,
  `Update to current view`, `Delete`.
- Prompts: `Name this filter`, `Rename filter` / delete confirm as in M6.

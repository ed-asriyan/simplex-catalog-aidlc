**Collaborator:** aidlc-quality-agent

## Contribution

Reviewed `stories.md` against `requirements.md` from the testability angle, with
the team TDD posture in mind (Vitest + `@testing-library/svelte`, 80% line
coverage on new/changed code, every AC objectively verifiable by an automated
test). The draft is strong: nearly every AC is a clean Given/When/Then with an
observable outcome. The observations below are gaps and refinements where a
testable case is missing, ambiguous, or asserts two things at once.

### Coverage gap — NFR-5 (graceful storage failure) has no story-level AC

`NFR-5` (page still renders and functions with presets only when `localStorage`
is unavailable or malformed; custom-filter reads/writes fail safe) is
user-observable and directly testable, yet no story or AC covers it. `US3.1`
covers the happy-path persistence only. This is the most important gap: the
malformed/absent-storage path is exactly where a silent failure would break the
servers list, and the construction guardrails forbid silent failures at the
storage boundary. Suggest a new story (e.g. **US3.4 — Survive unusable
storage**) or added ACs on `US3.1`:
- Given `localStorage` is unavailable (throws on read/write), When the page
  loads, Then the servers list and all five presets render and work, and no
  error is surfaced to the user.
- Given the custom-filters storage key holds malformed/non-parseable JSON, When
  the page loads, Then custom filters are treated as empty (presets still work)
  and the malformed value does not break the list or throw.
- Given `localStorage` write fails when saving a custom filter, When the user
  saves, Then the failure is surfaced/handled without breaking the current view
  (fail-safe, no uncaught error).

### Missing negative/edge ACs to add

- **US2.1 — empty (and whitespace-only) custom-filter name.** `AC2.1.1`
  covers the happy path but no AC defines behaviour for an empty/blank name.
  `AOQ-3` defers *duplicate* names to mockups, but empty-name is a distinct
  validation case and a natural TDD unit test. Suggest **AC2.1.4**: Given the
  "save current filter" flow, When the entered name is empty or whitespace-only,
  Then no custom filter is created (save is rejected/disabled) and the existing
  buttons are unchanged.
- **Empty result set for an applied view.** No AC covers a preset or custom
  filter that matches zero servers (e.g. **High Uptime** when no server has
  `uptime90 ≥ 90`). Suggest an AC (on `US1.2` or `US5.1`): Given a preset is
  applied that matches no servers, Then the list renders an empty result (no
  error/crash) and the preset is still shown as selected (`AC5.1.1` should hold
  even with zero matches, since selection is derived from the view, not the
  result count).
- **US4.1 — negative analytics path (ad-hoc apply emits nothing).** `FR-12`
  scopes analytics to preset and custom-filter applies only. The stories assert
  the positive emissions (`AC4.1.1`, `AC4.1.2`) but not the negative: applying
  an ad-hoc filter via the normal controls must NOT emit an apply event.
  Suggest **AC4.1.5**: Given analytics is configured, When I change the view via
  the normal ad-hoc filter/sort controls (not a preset/custom button), Then no
  preset/custom apply event is emitted.
- **US3.1 / US2.2 — custom-filter round-trip after reload.** `AC3.1.1` asserts
  the buttons are still listed after reload, but no AC asserts that *applying* a
  custom filter after reload yields the identical view. Suggest an AC: Given a
  saved custom filter, When I reload and then click it, Then the resulting URL
  and result set are identical to before the reload (definition round-trips
  intact).

### Refinements for objective verifiability

- **AC1.2.5 — uptime boundary + open threshold.** The `≥ 90` boundary needs an
  explicit test case: a server with `uptime90 = 90` is included and one with
  `uptime90 = 89.9…` is excluded. Also flag that this AC hardcodes the
  filter-plus-sort interpretation while `AOQ-5` is still open on whether "High
  Uptime" is sort-only (no threshold). If `AOQ-5` resolves to sort-only, `AC1.2.5`
  must change; recommend the AC note the dependency so a test is not written
  against a still-open decision.
- **AC5.1.5 — asserts two things at once.** It bundles "keyboard-operable" and
  "exposes selected/active state to AT" (`NFR-4`). For precise TDD, split into
  two ACs: (a) each preset/custom button is focusable and activatable via
  keyboard (Enter/Space applies the view); (b) the active button exposes its
  selected state to assistive technology (e.g. `aria-pressed`/`aria-selected`),
  and inactive buttons do not.
- **AC2.3.2 — "results are unaffected" is vague.** Sharpen to an observable
  equality: Given I delete the custom filter whose view is currently active,
  Then the rendered result set and the URL remain identical (only the saved
  definition is removed). Also worth a follow-on for `US5.1`: after deleting the
  active custom filter, the selected-state re-derives from the still-active view
  (it may now match a preset, or nothing).
- **Selected-state matching must include sort.** `AC3.2.4` correctly requires
  URLs to differ by sort (All Online vs Recently Added). The parallel is missing
  for selection: `AC5.1.1` should make explicit that a view sorted by
  `last_check` desc selects **All Online** while the same filter sorted by
  `created_at` desc selects **Recently Added** — i.e. matching is over the full
  view (filter + sort), so the two are never both/wrongly selected.
- **AC1.1.2 — "immediately".** The testable substance is "no confirmation
  step / no intermediate dialog before the list updates." Keep the observable
  ("no confirm step") as the assertion and treat "immediately" as descriptive,
  since wall-clock timing is not a stable unit-test assertion here.

### Notes (no change requested)

- `NFR-1` (store-vs-component layering) and `NFR-3` (no backend change) are
  architectural/structural and are appropriately enforced via test *location*
  and the absence of query changes rather than a user-facing AC; `AC1.2.6`
  already covers the no-backend-change guarantee at the story level.
- `AC3.3.2` (all existing controls continue to work) is a regression obligation
  best served by keeping/adding regression tests around the existing
  `updateFilter`/`updateSort` paths; it is testable but broad, so it should be
  read as "existing behaviour is covered by regression, not a single new test."

## Positions

OBJECT: FR/NFR coverage at the story level is not complete — NFR-5 (graceful
handling of unavailable/malformed `localStorage`) has no story or acceptance
criterion, despite being user-observable, directly testable, and a
silent-failure risk at the storage boundary. Add a story or ACs (see above)
before the set is signed off.
AGREE: With that gap closed and the empty-name, empty-result, and
negative-analytics ACs added, the remaining acceptance criteria are testable as
written.

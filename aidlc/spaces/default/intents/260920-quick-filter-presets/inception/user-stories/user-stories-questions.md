# User Stories — Plan & Questions

## Sources

- Requirements: `../requirements-analysis/requirements.md`
- Team practices: `../practices-discovery/team-practices.md`

## Story plan (proposed)

- **Persona development approach** — One unauthenticated actor class, the
  **catalog visitor**, modelled across a usage spectrum. Proposed two personas:
  a **power/dogfooding user** (switches views constantly, saves their own
  filters) and a **casual visitor** (occasional, uses presets, rarely saves).
- **Story format** — "As a [persona], I want [goal], so that [benefit]" with
  Given/When/Then acceptance criteria (per the inception phase rule). INVEST
  applied; stable `US{group}.{seq}` and `AC{group}.{seq}.{n}` IDs.
- **Story prioritization** — MoSCoW. Note: Delivery Planning is SKIP in this
  scope, so these priorities are the de-facto build ordering, not just an input
  to a later MVP decision.
- **Breakdown approach** — Proposed **by workflow area**: (1) Presets, (2)
  Custom-filter lifecycle, (3) Persistence & URL round-trip, (4) Analytics,
  (5) Selected-state feedback.

## Q1. Persona granularity — how many personas?

- A. Two personas — power/dogfooding user + casual visitor (captures the "I
  switch views constantly" dogfooding driver vs. the occasional browser)
- B. One persona — a single "catalog visitor" (simplest; requirements already
  model one actor)
- X. Other (please specify)

[Answer]: B. One persona — a single "catalog visitor" actor, matching how requirements already model it. The dogfooding/power-use behaviour is captured within that one persona's goals rather than as a separate persona.

## Q2. Story breakdown approach

- A. By workflow area (Presets / Custom-filter lifecycle / Persistence & URL /
  Analytics / Selected-state feedback)
- B. By persona (power user stories vs casual visitor stories)
- C. By epic (one epic "Quick filters", flat story list under it)
- X. Other (please specify)

[Answer]: A. By workflow area (Presets / Custom-filter lifecycle / Persistence & URL / Analytics / Selected-state feedback).

## Q3. Priority of the non-core areas (analytics, edit-in-place, rename)

The core (apply presets, save/apply/delete custom filters, URL round-trip,
persistence) is clearly Must Have. How should the rest be prioritized?

- A. Analytics = Should Have; rename + edit-in-place = Should Have; everything
  else Must Have (ship the whole feature this intent, nothing dropped)
- B. Analytics = Could Have (nice-to-have this intent); rename/edit-in-place =
  Should Have
- X. Other (please specify)

[Answer]: A. Ship the whole feature this intent — analytics = Should Have; rename + edit-in-place = Should Have; everything else Must Have. Nothing is dropped from this intent.

## Ambiguity analysis

No vague language, contradictions, or missing details in the three answers. All
map cleanly to the confirmed requirements (single actor per requirements'
Actors section; workflow-area breakdown mirrors the FR groupings; MoSCoW with
nothing deferred aligns with the confirmed "ship the whole feature" scope). No
follow-up questions required.

## Consolidated Summary Confirmation

The user-stories artifacts resolve to: **1 persona** (Catalog Visitor, spanning
casual → power/dogfooding use); **14 stories in 5 workflow groups** — Presets
(US1.1–US1.3), Custom-filter lifecycle (US2.1–US2.6), Persistence & URL
(US3.1–US3.3), Analytics (US4.1), Selected-state feedback (US5.1); **61
acceptance criteria** in Given/When/Then form. MoSCoW: core = Must Have,
analytics + rename + edit-in-place = Should Have (nothing dropped). Every FR/NFR
is traced in `traceability.json` (deterministic sensor passed; FR8 withdrawn,
NFR1/NFR2 process-N/A, NFR3 via AC1.2.6). Following review feedback, two explicit
"modify a facet after applying a view" stories were added — **US1.3** (change a
field after a preset) and **US2.6** (change a field after a custom filter): in
both, the change produces an ad-hoc view and the preset/custom deselects; for a
custom filter the saved definition is NOT auto-modified (persist only via
edit-in-place US2.5 or save-as-new US2.1), and re-clicking restores the saved
view. The mob (design, developer, quality) and the advisory product-lead review
contributed gaps that were folded in: storage fail-safe, empty-name validation,
empty-result handling, negative analytics path, save discoverability, stable
custom-filter id, delete confirmation, single coherent filter+sort apply, and
split accessibility ACs.

[Answer]: Looks correct

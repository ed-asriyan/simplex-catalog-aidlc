**Collaborator:** aidlc-design-agent

## Contribution

Reviewed from the UX / persona-fidelity angle. The draft is well-structured and the
apply/switch/persist/URL flows are strong. My observations focus on gaps in
discoverability, empty-state, destructive-action safety, and accessibility scope —
all at story/AC altitude, deferring visual treatment to Refined Mockups.

### 1. Discoverability of "save current filter" (US2.1) is assumed, never a requirement
AC2.1.1 begins "When I choose 'save current filter'" — it presumes the affordance
exists and is findable, but no story or AC establishes that the save entry point is
*discoverable* to the user. For the occasional-browser end of the persona spectrum,
"I didn't know I could save this" is the most likely usability failure. Suggest adding
an AC to US2.1, e.g.:
- **AC2.1.4 (proposed)** — Given I have an ad-hoc view configured via the normal
  controls, Then a "save current filter" affordance is available/visible in the filter
  area so I can save the current view (its exact placement/label is a Refined Mockups
  concern).

This also connects cleanly to the selected-state logic: when AC5.1.3 fires (no button
selected because the view is hand-edited), that is precisely the moment the save
affordance is most relevant — the "nothing selected" state is the natural cue that this
view is unsaved.

### 2. No empty-state story (zero custom filters yet)
Every custom-filter story (US2.1–US2.5) implicitly assumes at least one custom filter,
but a first-time visitor sees only the preset row. There is no AC describing what the
button area looks/behaves like before any custom filter exists — i.e. that presets
render normally and the feature degrades gracefully to "presets only" (which also aligns
with NFR-5's fail-safe posture). Suggest an AC, e.g.:
- **AC2.1.5 (proposed)** — Given I have never saved a custom filter, Then the filter
  area shows the presets and the save affordance, and no empty/broken custom-filter
  region is shown.

### 3. Discoverability of rename / edit-in-place / delete affordances (US2.3–US2.5)
US2.3/US2.4/US2.5 jump straight to "When I delete it / rename it / re-save over it" but
no AC establishes that each custom-filter button *exposes reachable management controls*.
Since a custom filter is primarily an apply-on-click button (US2.2), the manage actions
must be reachable without triggering an apply. Suggest one shared AC (attach to US2.2 or
a new umbrella AC), e.g.:
- **AC (proposed)** — Given a saved custom filter, Then its rename / edit-in-place /
  delete actions are reachable via a control that does not itself apply the filter
  (avoiding an accidental view-replace when the user meant to manage it).

### 4. Delete is destructive with no confirmation or undo (US2.3)
Presets intentionally apply with no confirm (FR-1/AC1.1.2) — correct for a reversible
action. Delete is different: it permanently removes a user's saved definition, and unlike
an accidental preset click it cannot be reversed by clicking again. Per the
error-prevention principle, a destructive irreversible action warrants either a
confirmation step or an undo. This is currently silent in US2.3. Suggest:
- **AC2.3.3 (proposed)** — Given I trigger delete on a custom filter, Then the deletion
  is guarded against accidental loss (a confirm step or an undo affordance — exact
  pattern deferred to Refined Mockups).

### 5. Name entry has no validation AC (US2.1)
AC2.1.1 says "enter a name" but nothing covers an empty / whitespace-only name.
Duplicate-name handling is legitimately deferred (AOQ-3), but empty-name is a distinct,
uncovered user-facing case. Suggest:
- **AC2.1.6 (proposed)** — Given the save dialog, When I attempt to save with an empty or
  whitespace-only name, Then the save is prevented and the reason is communicated (no
  unnamed/blank button is created).

### 6. Default-load selected state should be made explicit (US3.3 + US5.1)
AC3.3.1 sets the no-URL default view to `status = online`, sort `last_check` desc — which
is byte-for-byte the **All Online** preset definition (AC1.2.1). By AC5.1.1 that means the
All Online preset will render as *selected* on a cold load. This is good and consistent,
but it is currently only inferable by cross-reading two ACs. Recommend an explicit AC so
it is a deliberate, testable behavior rather than an accident:
- **AC5.1.6 (proposed)** — Given I open the servers page with no view in the URL, Then the
  default view is shown (AC3.3.1) and, because it matches All Online, the All Online preset
  is shown as selected.

### 7. Accessibility (NFR-4) coverage is narrower than the feature surface
AC5.1.5 is good but scopes keyboard-operability + AT state to the **preset/custom apply
buttons only**. NFR-4 and inclusive-design practice should also cover the *management*
surface introduced by Group 2: the save affordance, the name-entry dialog, and the
rename/edit/delete controls. As written, a keyboard-only or screen-reader user is
guaranteed accessible *apply* but not accessible *save/manage*. Suggest broadening, e.g.:
- **AC5.1.7 (proposed)** — Given the save affordance, the name-entry input, and the
  rename/edit/delete controls, Then each is keyboard-operable and labelled for assistive
  technology (NFR-4), consistent with existing UIkit usage.

### 8. Overflow behavior with unbounded custom filters (FR-7, US2.1)
FR-7 mandates "no hard limit" on saved custom filters, rendered as buttons alongside
presets. A single row with unbounded buttons has a real layout/scannability consequence
(wrap vs. horizontal scroll vs. overflow menu). This is primarily a Refined Mockups
concern and I am *not* proposing a story here — flagging it so mockups deliberately
handles the many-filters case rather than assuming a short row.

### Personas
The single Catalog Visitor persona with an in-persona spectrum (occasional browser vs.
power/dogfooding user) is the right call and is well-justified against Q1 and the
requirements' single Actor. It captures the real usage range. One usage mode worth
noting for completeness: the *recipient* of a shared/bookmarked URL (someone arriving at
a URL-encoded view, US3.2/AC3.2.2) — functionally covered, and it does not warrant a
separate persona, but the persona's "share or bookmark" goal is currently sender-side only.

## Positions

AGREE: The apply/switch model (US1.1, US1.2, US2.2) — immediate, whole-view-replace, mutually-exclusive presets — is clear, correct, and matches the requirements.
AGREE: The single-persona-with-spectrum decision (P1) faithfully models the real user range and is well-justified.
AGREE: Selected-state precedence and the "no match = nothing selected" behavior (US5.1, AC5.1.1–AC5.1.4) is sound and unambiguous.
OBJECT: NFR-4 accessibility is under-covered — AC5.1.5 scopes keyboard/AT support to the apply buttons only, leaving the save affordance, name-entry dialog, and rename/edit/delete controls (all new in Group 2) with no accessibility AC, so a keyboard/screen-reader user is not guaranteed to be able to save or manage filters. This is a user-facing, testable gap (see observation 7).
OBJECT: The custom-filter surface has no discoverability or empty-state coverage — no AC establishes that "save current filter" is findable, that management actions are reachable without applying, or how the area behaves before any custom filter exists. A first-time / occasional visitor may never discover the feature exists. These are testable story/AC-altitude gaps (see observations 1, 2, 3).

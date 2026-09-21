# User Stories Assessment

## Decision

**Execute.**

## Rationale

The quick-filter-presets feature is squarely user-facing: it adds interactive
UI (preset buttons, custom-filter create/edit/delete, selected-state
indication) that changes how a person operates the servers page. Behaviour
depends on user intent (which view they want, saving/naming their own filters),
so acceptance criteria expressed as user stories add real value for both design
(Refined Mockups) and testing (TDD acceptance tests) downstream.

## Factors considered

- **Project type**: front-end UI feature on an existing Svelte app — user-facing.
- **User-facing scope**: preset buttons, custom-filter CRUD, persistence,
  URL-linkable views, selected-state feedback — all directly experienced by users.
- **Complexity signals**: multiple interaction flows (apply / save / rename /
  edit / delete), a derived tor/clearnet distinction, and view (filter+sort)
  semantics that must round-trip through the URL. These benefit from
  story-level acceptance criteria.
- **Personas**: a single unauthenticated actor class (catalog visitor) with a
  usage spectrum from casual browser to power/dogfooding user who switches views
  frequently — enough persona nuance to justify stories, not so much as to need
  a large persona set.

## Key areas where stories add the most value

1. Applying presets (replace-whole-view semantics, mutual exclusivity, selected state).
2. Custom-filter lifecycle (save current view, rename, edit-in-place, delete).
3. Persistence and URL round-trip (localStorage durability, linkable views).
4. Analytics on apply (measurability of the release goal).

**Collaborator:** aidlc-quality-agent

## Contribution

Independent verification of the frontend testing evidence, plus a QA-lead
recommendation set for the human interview. I re-inspected
`simplex-catalog-frontend/package.json`, searched the whole `src/` tree for
`*.test.*`/`*.spec.*` files, and read `CI.yml`/`build.yml` directly.

**Confirmed facts (matches the lead's evidence, independently verified):**
- `package.json` `devDependencies` contain no test framework or runner —
  no `vitest`, `jest`, `@testing-library/*`, `playwright`, `@testing-library/svelte`,
  or `c8`/`istanbul` coverage tool — and no `test` script exists.
- A recursive search of `simplex-catalog-frontend/src` for
  `*.test.*`/`*.spec.*` returned **zero files**. There is no existing test
  suite of any kind (unit, component, or e2e) to be compatible with.
- `CI.yml` runs only the reusable `build.yml` job (checkout → write `.env` →
  `make prod_build_bundle` → upload Pages artifact) on `pull_request`. No
  lint, type-check, or test step is present anywhere in the pipeline.
  `svelte-check` exists as an `npm run check` script but is not invoked by
  any workflow.
- Stack facts relevant to tooling choice: Svelte 5.56 (`svelte5-router`
  confirms Svelte 5 runes-era APIs are in play), Vite 8, TypeScript 6,
  `@sveltejs/vite-plugin-svelte` 7.2 — i.e. a Vite-native build, not
  SvelteKit's full-stack framework (no `@sveltejs/kit` dependency despite
  the `svelte-kit sync` call in `check`, which is a vestige of scaffolding
  used only for TS sync, not routing/SSR).
- The codebase has a clear service/store split (`src/store/{bots,relays,
  servers}/*-service.ts`, `*-store.ts`, plus `abstract-store.ts` and
  `src/utils.ts`) and filter-relevant logic already lives in
  `servers-service.ts`, `countries-service.ts`, and several `*/list.svelte`
  / `table/index.svelte` components. This is directly relevant to
  "quick-filter-presets": the feature will likely add filter-preset
  state/logic to the store layer and UI to Svelte components — both need
  a story since there is currently no harness for either.

**Recommendation — test framework (for the interview, not yet decided):**
- **Vitest** is the natural fit here, not merely "no evidence against it":
  it shares Vite's config/transform pipeline (same `vite.config` resolves
  aliases/plugins for both dev server and tests, no separate Babel/webpack
  config to maintain), has first-class Svelte 5 support via
  `@testing-library/svelte` (v5-compatible) or `vitest-browser-svelte` for
  runes-aware component tests, and is the de facto standard for
  Vite+Svelte/SvelteKit projects as of 2026 — this keeps the toolchain
  small for a solo/AI-driven project rather than introducing Jest's
  separate transform/module-mocking stack alongside Vite.
- Recommend layering: `vitest` for store/service/util unit tests (pure
  TS, no DOM needed — fast), `@testing-library/svelte` + `vitest`'s
  `jsdom`/`happy-dom` environment for component-level tests of filter UI
  (e.g. a `QuickFilterPresets.svelte` component), and treat full
  browser/e2e (Playwright) as **out of scope for this feature-sized
  change** unless the team already plans e2e investment — recommend
  raising that as a separate, later decision rather than bootstrapping
  Playwright for a single feature's practices-discovery gate.
- Coverage tooling: Vitest's built-in `@vitest/coverage-v8` (zero extra
  config) rather than a separate `nyc`/`istanbul` setup.

**Recommendation — coverage floor:**
- Org default is 80% line coverage for `feature`/`mvp` scopes with CI
  execution before merge, and nothing in the evidence justifies deviating
  downward — this is a small solo/AI-driven project, but the org floor is
  itself already calibrated as a floor, not a maximalist target, and 80%
  line coverage on new filter-preset code (store + component) is
  realistic to hit test-after with Vitest given the codebase's existing
  service/store separation (pure functions/stores are easy to hit high
  coverage on; DOM-heavy `.svelte` view code is harder — recommend scoping
  the floor to **statements/lines** rather than branches for this pass to
  keep the bar achievable, and applying it to **new/changed code**
  introduced by this feature rather than retroactively to the entire
  pre-existing untested codebase, since retroactively demanding 80% across
  every existing store/component would turn a filter-preset feature into a
  test-the-whole-app project).
- If the team wants a stricter or looser floor, that is exactly what this
  interview should surface and record explicitly in `team.md` under
  `Testing Posture` — I have no evidence indicating the org default is
  wrong for this project, only that it has never been affirmed here.

**Recommendation — CI gating:**
- Yes, CI should gate PR merges on tests passing once a test suite exists.
  Concretely: add a `test` step to `CI.yml` (or a new job called from
  `build.yml`'s pull_request trigger) running `vitest run` (non-watch
  mode) before/alongside the existing build step, and fail the PR check
  if it does not pass. This is a small, low-risk addition to the existing
  `pull_request`-triggered `CI.yml` and directly implements the org
  default of "CI execution before merge."
- While this workflow is in scope: `svelte-check` (already available as
  `npm run check`) is currently unused by CI despite existing as tooling.
  Recommend wiring it into the same PR gate (or noting explicitly that the
  team defers this) — a type-check gate is cheap insurance for a
  Svelte 5 + TypeScript codebase and is unrelated to the test-framework
  decision, so it should not block choosing Vitest.
- No auto-scaling/load-testing or NFR validation applies at this stage —
  this is a small static-site frontend (GitHub Pages deploy target) with
  no server-side component of its own being touched by this feature;
  performance/NFR validation is out of scope for practices-discovery here.

**Gaps the interview must resolve (in addition to the lead's open items):**
1. Confirm the test framework choice explicitly (Vitest recommended) —
   do not let it default silently at Code Generation.
2. Confirm the coverage floor's scope: applied to new/changed code only
   for this feature, or retroactively expected across the pre-existing
   untested codebase (I recommend new/changed only, given zero prior
   coverage).
3. Confirm whether `svelte-check` becomes a required CI gate alongside
   tests, or stays a manual/local-only check.
4. Confirm whether component tests (DOM-level, via
   `@testing-library/svelte`) are in scope for this feature's UI pieces,
   or whether store/service-level unit tests alone satisfy the floor for
   this pass — this affects effort estimation at Delivery Planning.
5. Methodology (TDD/BDD/ATDD/test-after): the lead correctly leaves this
   open; from a QA standpoint test-after (the org default) is a reasonable
   starting point for a project with zero existing test infrastructure,
   since introducing both a framework and a TDD discipline in the same
   feature raises risk — but this should still be an explicit human
   answer, not an inherited default.

## Positions

AGREE: The draft correctly treats Testing Posture as fully open rather than assuming Vitest or any framework has already been chosen — matches my independent evidence.
AGREE: The Deployment section's narrower-than-org-default characterization (single GitHub Pages environment, no staging, no manual gate, no smoke test) is accurate per my own read of `CD.yml`/`build.yml`.
AGREE: The Code Style section's "no linter/formatter config, `svelte-check` unused by CI" finding is confirmed by my own package.json/CI inspection.
OBJECT: The draft's Testing Posture section lists framework choice as fully open with "no evidence in the repo favors one over another" — I'd sharpen this for the interview: the Vite-native build (no SvelteKit) plus Svelte 5 make Vitest a clear default recommendation to present to the human, not a coin flip, so the interview should be framed as "confirm Vitest" rather than "pick a framework from scratch."

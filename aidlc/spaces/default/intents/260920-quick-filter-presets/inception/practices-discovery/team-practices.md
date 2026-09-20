# Team Practices — DRAFT

> **DRAFT — pending the human interview.** This mirrors the five headings
> of `aidlc/spaces/default/memory/team.md` and proposes starting points
> derived from the org-level defaults and the brownfield inspection in
> `evidence.md`. Nothing here is affirmed; the practices-discovery
> interview confirms, edits, or rejects each item before anything is
> promoted to `team.md`. This is the first practices-discovery run for
> this project — `team.md` is currently template-empty across all five
> headings.

## Way of Working

- **Proposed starting point (org default, not yet confirmed against
  frontend-submodule history):** trunk-based development. Work merges to
  `main`/`master` via short-lived feature branches. We squash-merge Bolt
  branches into the trunk, one commit per Bolt named by the Bolt slug.
- **Evidence basis:** the umbrella repo's current branch
  (`claude/sharp-lamport-jy3q3f`) is a single short-lived branch off
  `master`, consistent with this pattern, but the frontend submodule's
  own commit/PR history was not separately inspected, so this is
  inherited from `org.md` rather than confirmed from repo evidence. See
  `evidence.md` § 1.
- **Open for interview:** does the team want to affirm this as-is, or
  specialize it (e.g., branch naming convention, PR review requirement)?

## Walking Skeleton

- **Proposed starting point (org default):** governed by the active
  scope file's `skeleton: on|off` declaration, not by a fixed team rule.
  No repo evidence bears on this either way.
- **Open for interview:** none required unless the team wants a
  project-specific override of when a skeleton Bolt runs.

## Testing Posture

- **NOT proposed — this is exactly what this stage's interview needs to
  resolve.** The frontend submodule currently has **no test framework
  dependency** (no vitest/jest/testing-library in `package.json`) and
  **no CI test step** (`CI.yml` → `build.yml` runs only a build). There
  is nothing existing to be compatible with, so the interview must
  choose:
  - **Methodology**: TDD, BDD, ATDD, or test-after (org default when
    unaffirmed is test-after).
  - **Test framework/tooling**: e.g., Vitest (natural fit for the
    existing Vite/Svelte toolchain) vs. another option — no evidence in
    the repo favors one over another; this must come from the human.
  - **Ordering** and **coverage floor**: org default is "implement each
    testable layer, then write and run that layer's tests," with an 80%
    line-coverage floor for `feature`/`mvp`/etc. scopes and CI execution
    before merge — but CI does not currently run any test step, so
    wiring a test step into `CI.yml`/`build.yml` is itself part of what
    the interview should confirm.
  - See `evidence.md` § 3 for the full absence-of-tooling evidence.

## Deployment

- **Proposed starting point, adjusted for what the pipeline evidence
  actually shows (narrower than the plain org default):** deploy
  automatically on push to `master` to the project's single current
  environment (GitHub Pages), via the existing `CD.yml` /
  `build.yml` reusable-workflow pair. There is currently **no separate
  staging environment**, **no manual production-approval gate**, and
  **no post-deploy smoke test** in the pipeline as written.
- **Evidence basis:** `CD.yml` triggers on `push` to `master`, runs
  `build` then `deploy` (`actions/deploy-pages@v5`) with no
  `environment:` protection rule and no smoke-test step; `CI.yml` runs
  build-only on `pull_request`. See `evidence.md` § 2.
- **Open for interview:** the org default calls for deploy-on-merge to a
  *staging* environment with a separate manual gate before production.
  The team should confirm whether the current single-environment/
  no-gate GitHub Pages deploy is the intended, affirmed practice for
  this project (e.g., because there is currently only one environment
  and low risk), or whether a staging tier / approval gate / smoke test
  should be added as the project matures.

## Code Style

- **Proposed starting point (org default, currently nothing to defer
  to):** defer to project-level linter/formatter configuration once one
  exists. Today, no `.prettierrc*` / `.eslintrc*` config and no
  `eslint`/`prettier` dependency exist in the frontend submodule;
  `svelte-check` (TypeScript/Svelte type-checking) exists as a script
  (`npm run check`) but is not run in CI.
- **Evidence basis:** see `evidence.md` § 3.
- **Open for interview:** should a linter/formatter be adopted now (and
  if so, wired into CI), or does the team want to keep deferring until a
  config is introduced organically? Should `svelte-check` be added as a
  CI gate?

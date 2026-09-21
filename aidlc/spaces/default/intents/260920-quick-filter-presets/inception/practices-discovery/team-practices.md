# Team Practices

> Affirmed at the practices-discovery interview for this project. These five
> sections mirror the headings of `aidlc/spaces/default/memory/team.md` and are
> written in the team's affirmed voice — every item below was confirmed, edited,
> or set by the human at the interview gate (see
> `practices-discovery-questions.md` for the filled answers and
> `evidence.md` for the brownfield inspection they were decided against). This
> is the first practices-discovery run for this project; before it, `team.md`
> was template-empty across all five headings.

## Way of Working

We develop on **short-lived feature branches, squash-merged** into the trunk.
For this project that base practice is refined by a fuller **multirepo
branching, merge, and finishing policy**, because this is a git-submodule
multirepo (an umbrella repo plus sub-repos), and it is authoritative here:

- **Every AI-DLC intent is developed on its own feature branch(es), never
  directly on `master`.** Each new intent starts fresh feature branch(es);
  branches are not reused across intents.
- **Matching branch names across every affected repo.** When a feature touches
  multiple sub-repos, the *same* feature-branch name is used in every affected
  sub-repo. The umbrella repo (this repo) also gets its own feature branch of
  the same name, because updating the submodule pointers requires a commit
  here too.
- **The AI never merges to `master` and never deploys to production.** Merging
  the feature branches into `master` is a human-only action, done manually at
  the human's own discretion once they are satisfied — including the umbrella
  repo's feature branch with its updated submodule pointers.
- **Two required finishing steps at the end of every intent, both of them:**
  1. **Commit and push** the intent's work to the feature branch(es) in every
     affected repo, including this umbrella repo's feature branch with the
     updated submodule pointers.
  2. **Deploy the complete system locally** (all components, built from the
     feature branches) to verify it works end-to-end before finishing.
  These are complementary, not alternatives — both the push and the local
  full-system deployment must happen before the intent is considered done.
- We **squash-merge** feature branches so the trunk keeps a clean linear
  history; the full branch history is preserved on the source branch until the
  human discards it.

See `## Deployment` for how this policy interacts with the GitHub Pages
auto-deploy that master already performs.

## Walking Skeleton

We **always build a thin end-to-end slice (a walking skeleton) first**,
regardless of feature size — including small, seemingly self-contained
features like this one. The skeleton runs the whole way through and proves the
pieces connect before the real feature work goes in. This is a standing team
practice, not a scope-file toggle: it applies to every intent.

## Testing Posture

We treat tests as a first-class deliverable in every intent, written with a
**test-first (TDD)** discipline.

- **Methodology**: tdd
- **Ordering**: For each testable layer, write a failing test first, then
  implement just enough production code to make that test pass before moving on.
- **Framework/tooling**: **Vitest** (reuses the existing Vite/Svelte 5 build
  config), with **@vitest/coverage-v8** for coverage and
  **@testing-library/svelte** for Svelte 5 component-level tests. Store/service/
  util logic is covered with plain Vitest unit tests; UI components are covered
  with `@testing-library/svelte` under Vitest's `jsdom`/`happy-dom` environment.
  Full browser/e2e (Playwright) is out of scope for this feature and is a
  separate, later decision.
- **Coverage floor**: **80% line coverage on new/changed code only** —
  introduced by the current intent, not retroactively across the pre-existing
  untested codebase. The floor is scoped to statements/lines.
- **CI gating**: CI **must fail a pull request if the tests do not pass**. Tests
  run in non-watch mode (`vitest run`) as a required PR check, wired into the
  existing `pull_request`-triggered `CI.yml` pipeline.
- **`svelte-check` is a required CI check** as well, gating merges alongside the
  test run (it exists today as `npm run check` but was not previously CI-wired).

Coverage floors and affirmed quality targets may not be weakened to make a step
pass.

## Deployment

`master` **keeps auto-deploying to GitHub Pages** on every push, exactly as it
does today (via the existing `CD.yml`/`build.yml` pair, `actions/deploy-pages`).
There is **no persistent staging environment** and no separate manual
production-approval job in the pipeline; the single GitHub Pages target is the
affirmed deployment posture for a project of this size.

Deployment is bounded by the multirepo Way of Working above:

- The AI **never triggers a production deploy** and **never merges to
  `master`** — production deploy happens only as a consequence of the human's
  own manual merge of the feature branches into `master`, which then fires the
  existing GitHub Pages auto-deploy.
- The AI's own deployment obligation is the **end-of-intent local full-system
  deployment** (all components, built from the feature branches) used to verify
  the change end-to-end before finishing — not any push to the live GitHub
  Pages site.

## Code Style

We **adopt ESLint + Prettier now and wire them into CI** (the project had no
linter or formatter config before this stage). Linter/formatter failure blocks
the PR, alongside the test and `svelte-check` gates.

We also affirm the following existing, previously-unwritten conventions as team
practice, so new code (including the quick-filter-presets feature) has a
concrete standard to follow — these are structural/naming conventions a linter
does not enforce:

- **Filenames**: kebab-case for both `.ts` and `.svelte` files.
- **Folder entry component**: a folder's entry component is always named
  `index.svelte`.
- **Component props**: a local `interface Props { ... }` declared directly above
  the `$props()` destructure in each Svelte component.
- **Boundary mapping**: camelCase ↔ snake_case mapping is done in the service
  layer; database snake_case field names never leak past the service into the
  domain types, stores, or components.
- **Feature-based layering**: `src/store/<feature>/` holds service + state;
  `src/components/<feature>/` holds presentation. The two are kept separate, and
  new feature logic (e.g. filter-preset state) lands in the store layer, not in
  component-local logic.
- **Two-tier nesting**: a multi-file subfeature gets its own directory with
  `index.svelte` as the entry point and sibling files for its internal pieces;
  small reusable leaf components shared across a feature live flat in a
  `fields/`-style directory.
- **Import paths**: the `@/*` alias (→ `src/*`) is used for cross-feature
  imports; relative paths are used for same-directory imports.

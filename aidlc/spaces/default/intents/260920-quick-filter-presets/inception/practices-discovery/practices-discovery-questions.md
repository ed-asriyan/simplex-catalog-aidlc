# Practices Discovery — Interview

## Sources

- Lead draft: `inception/practices-discovery/team-practices.md`, `evidence.md`
- Contributions: `inception/practices-discovery/contributions/aidlc-quality-agent.md`, `aidlc-developer-agent.md`, `aidlc-devsecops-agent.md`

This is the first practices-discovery run for this project (`team.md` is still empty). The frontend has no existing test framework, no linter/formatter, and a single-environment auto-deploy pipeline — so most of these questions are genuinely open rather than confirmations of something already in place.

## Q1. How should code move from a feature branch into the main branch?

The default (from your org-wide settings) is: work happens on short-lived feature branches, and each finished branch is squashed into one commit on `main`/`master`. Your repo's current single working branch is consistent with this, but doesn't prove it as a rule.

- A. Yes, that's the convention: short-lived branches, squash-merged into main
- B. Different convention — I'll specify
- X. Other (please specify)

[Answer]: A. Yes, that's the convention: short-lived branches, squash-merged into main — refined by Q7's fuller multirepo branching/merge policy below (merges to master are done by the human, not automatically)

## Q2. Build a thin end-to-end slice first? A walking skeleton is a minimal version that runs the whole way through, built first to prove the pieces connect before the real features go in.

For a small feature like this one (adding buttons and a filter-CRUD flow to an existing page), a walking skeleton is usually unnecessary — there's no new architecture to prove out.

- A. No — not needed for small features like this one; only use it when starting something architecturally new
- B. Yes — always build a thin slice first regardless of feature size
- X. Other (please specify)

[Answer]: B. Yes — always build a thin slice first regardless of feature size

## Q3. Which test framework should this project use, given it has none today?

Since the frontend is built directly on Vite (not the fuller SvelteKit framework) with Svelte 5, Vitest is the natural fit — it reuses the existing Vite build config, and works well with Svelte 5 via `@testing-library/svelte`.

- A. Vitest (recommended — fits the existing Vite/Svelte 5 setup)
- B. A different framework — I'll specify
- X. Other (please specify)

[Answer]: A. Vitest (recommended — fits the existing Vite/Svelte 5 setup)

## Q4. What testing approach should the team follow — write tests before or after the code, and using what style?

- A. Test-after (write the code, then write tests for it) — the safest default when there's no existing test culture to match
- B. Test-first / TDD (write a failing test, then the code to pass it)
- C. BDD-style (Given/When/Then scenarios written first)
- D. A mix — I'll specify
- X. Other (please specify)

[Answer]: B. Test-first / TDD (write a failing test, then the code to pass it)

## Q5. Should new code be required to hit a test-coverage target, and should CI block merges if tests fail?

Since the codebase has zero test coverage today, requiring a percentage across the *whole* app would mean testing everything, not just this feature.

- A. Yes to both — require 80% line coverage on new/changed code only (not retroactively on the existing untested code), and make CI fail a pull request if tests don't pass
- B. Require CI to run tests, but no specific coverage percentage
- C. Don't add a CI gate for this yet
- X. Other (please specify)

[Answer]: A. Yes to both — require 80% line coverage on new/changed code only (not retroactively on the existing untested code), and make CI fail a pull request if tests don't pass

## Q6. Should the existing type-checker (`svelte-check`) also become a required CI check, alongside tests?

It already exists as a script but nothing runs it automatically today.

- A. Yes — add it as a required CI check
- B. No — leave it as a manual/local check for now
- X. Other (please specify)

[Answer]: A. Yes — add it as a required CI check

## Q7. Your deployment today auto-publishes to GitHub Pages on every merge to `master`, with no staging environment and no approval step. Keep it that way, or add a safety net?

- A. Keep it as-is — one environment, auto-deploy on merge, no gate (appropriate for a project this size)
- B. Add a staging environment and/or a manual approval step before it's live
- X. Other (please specify)

[Answer]: X. Other (please specify) — Master keeps auto-deploying as today; no persistent staging environment. This also sets a fuller branching/merge policy for how AI-DLC work moves through this multirepo project:
  - Every feature (every AI-DLC intent) is developed on its own feature branch(es), never directly on master.
  - This project is a multirepo (git submodules): when a feature touches multiple sub-repos, the SAME feature-branch name is used in every affected sub-repo.
  - The umbrella multirepo itself (this repo) also gets its own feature branch of the same name, because updating submodule pointers requires a commit here too.
  - The AI never merges to master and never deploys to production. The AI-DLC flow ends when the human-approved changes are pushed to the feature branch(es) (including this umbrella repo's feature branch with updated submodule pointers).
  - At the end of each feature/intent, before finishing, the AI deploys the complete system (all components) locally on its own machine, built from the feature branches, to verify everything works end-to-end.
  - The human merges all the feature branches (including this umbrella repo's) into master manually, at their own discretion, once satisfied.
  - Each new intent starts fresh feature branch(es); branches are not reused across intents.

## Q7 Follow-up: Clarify the branching/deployment policy

The requester said "Close, but adjust" to a restated summary of the Q7 policy (feature branches per intent across every affected repo including this umbrella one, AI never merges to master or deploys to production, AI deploys the full system locally at the end of each intent, human merges to master manually). What specifically should change?

[Answer]:

## Q8. Should the project adopt a linter/formatter (e.g. ESLint + Prettier) now, given it has none today?

Separately from tooling: the codebase already has real, unwritten conventions worth naming explicitly — kebab-case filenames, a folder's entry component always named `index.svelte`, and a clear separation between the data/state layer (`src/store/<feature>/`) and the display layer (`src/components/<feature>/`). Naming these as team conventions doesn't depend on adopting a linter.

- A. Adopt a linter/formatter now and wire it into CI
- B. Keep deferring — no linter/formatter for now, but do affirm the existing naming/layering conventions as team practice
- C. Keep deferring on both — no linter and no explicit convention affirmation yet
- X. Other (please specify)

[Answer]: A. Adopt a linter/formatter now and wire it into CI

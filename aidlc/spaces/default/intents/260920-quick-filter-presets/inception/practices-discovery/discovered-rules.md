# Discovered Rules

> Hard constraints the human explicitly affirmed at this stage's interview.
> `## Mandated` holds `ALWAYS ...` rules and `## Forbidden` holds `NEVER ...`
> rules, matching the promotion format used in
> `aidlc/spaces/default/memory/project.md` — these are promoted verbatim into
> that file's `Mandated`/`Forbidden` sections at the gate. Every rule below
> traces to a filled `[Answer]` in `practices-discovery-questions.md`.

## Mandated

- ALWAYS develop each AI-DLC intent on its own feature branch(es), never
  directly on `master`, using the same feature-branch name across every
  affected sub-repo and in the umbrella repo (whose branch carries the updated
  submodule pointers), and start fresh branch(es) for each new intent rather
  than reusing them. (affirmed 2026-09-20)
- ALWAYS build a walking skeleton (a thin end-to-end slice that runs the whole
  way through) first, regardless of feature size. (affirmed 2026-09-20)
- ALWAYS finish an intent by doing both: (1) commit and push the work to the
  feature branch(es) in every affected repo including the umbrella repo, and
  (2) deploy the complete system (all components) locally to verify it works
  end-to-end. (affirmed 2026-09-20)
- ALWAYS write tests first (TDD): write a failing test, then implement just
  enough code to make it pass, for each testable layer. (affirmed 2026-09-20)
- ALWAYS meet an 80% line-coverage floor on new/changed code (not
  retroactively on the pre-existing untested codebase). (affirmed 2026-09-20)
- ALWAYS gate CI before merge on the tests passing and on `svelte-check`
  passing, and fail the pull request if either does not pass. (affirmed
  2026-09-20)

## Forbidden

- NEVER merge to `master` — merging feature branches into `master` is a
  human-only action. (affirmed 2026-09-20)
- NEVER deploy to production. (affirmed 2026-09-20)
- NEVER let database snake_case field names leak past the service layer into
  domain types, stores, or components; map them to camelCase at the service
  boundary. (affirmed 2026-09-20)

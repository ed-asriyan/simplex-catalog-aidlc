# Project-Level Rules

> Project-specific specialisation and corrections. Loaded after `org.md` and
> `team.md` as strict-additive guidance; contradictions with broader policy
> are rejected. Populated by practices-discovery and the self-learning loop.
>
> Use sparingly: most teams don't need a project layer. Reach for it
> only when this specific project needs stable, durable guidance beyond the
> team practice (for example, package-specific release checks or an additional
> regression suite for a legacy component).

## Way of Working

<!-- Project-specific specialisation. Example: -->
<!-- This monorepo requires package-scoped branch names and a package owner -->
<!-- review in addition to the team's normal merge policy. -->

## Walking Skeleton

<!-- Project-specific specialisation. Example: -->
<!-- The walking skeleton must exercise the legacy service adapter as well -->
<!-- as the new service boundary. -->

## Testing Posture

<!-- Project-specific specialisation. -->

## Change Control

<!-- Project-specific. Mode: strict or relaxed. Strict here holds for every intent and cannot be changed from chat. -->

## Deployment

<!-- Project-specific specialisation. -->

## Code Style

<!-- Project-specific specialisation. -->

## Tech Stack

<!-- Technology choices locked for this project. -->

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

## Scope Overrides

<!-- Custom scope rules for this project. -->

## Forbidden

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: NEVER [behavior] (affirmed [date]) -->
<!-- Example: NEVER throw exceptions across service layer boundaries (affirmed 2026-05-17) -->

- NEVER merge to `master` — merging feature branches into `master` is a (affirmed 2026-09-21)

human-only action. (affirmed 2026-09-20) (affirmed 2026-09-21)

- NEVER deploy to production. (affirmed 2026-09-20) (affirmed 2026-09-21)

- NEVER let database snake_case field names leak past the service layer into (affirmed 2026-09-21)

domain types, stores, or components; map them to camelCase at the service (affirmed 2026-09-21)

boundary. (affirmed 2026-09-20) (affirmed 2026-09-21)

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

- ALWAYS develop each AI-DLC intent on its own feature branch(es), never (affirmed 2026-09-21)

directly on `master`, using the same feature-branch name across every (affirmed 2026-09-21)

affected sub-repo and in the umbrella repo (whose branch carries the updated (affirmed 2026-09-21)

submodule pointers), and start fresh branch(es) for each new intent rather (affirmed 2026-09-21)

than reusing them. (affirmed 2026-09-20) (affirmed 2026-09-21)

- ALWAYS build a walking skeleton (a thin end-to-end slice that runs the whole (affirmed 2026-09-21)

way through) first, regardless of feature size. (affirmed 2026-09-20) (affirmed 2026-09-21)

- ALWAYS finish an intent by doing both: (1) commit and push the work to the (affirmed 2026-09-21)

feature branch(es) in every affected repo including the umbrella repo, and (affirmed 2026-09-21)

(2) deploy the complete system (all components) locally to verify it works (affirmed 2026-09-21)

end-to-end. (affirmed 2026-09-20) (affirmed 2026-09-21)

- ALWAYS write tests first (TDD): write a failing test, then implement just (affirmed 2026-09-21)

enough code to make it pass, for each testable layer. (affirmed 2026-09-20) (affirmed 2026-09-21)

- ALWAYS meet an 80% line-coverage floor on new/changed code (not (affirmed 2026-09-21)

retroactively on the pre-existing untested codebase). (affirmed 2026-09-20) (affirmed 2026-09-21)

- ALWAYS gate CI before merge on the tests passing and on `svelte-check` (affirmed 2026-09-21)

passing, and fail the pull request if either does not pass. (affirmed (affirmed 2026-09-21)

2026-09-20) (affirmed 2026-09-21)

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
- When a free-text answer to a structured question is already a complete, unambiguous substantive answer (not a request to discuss further), record it verbatim rather than re-prompting the user. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:intent-capture:d9ddd861a20bda0bed4495d91a50a356970674faa64dc18678053057156624fa -->
- When a stage's default question topics reference an upstream artifact type that this scope skips, drop those topics from the questions file rather than asking about an artifact that doesn't exist. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:approval-handoff:745e2f7f09e03d02a54790d170ba5bd17f95a9c42d1603a6127b711316575867 -->
- A `consumes` entry marked required with `consumes_absent` `expected: true` is absent by scope design, not a gap — proceed using only the artifacts that do exist rather than treating it as a missing dependency. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:approval-handoff:0d0d53bbb41f30686425c531e01637a8f8832a4358b4615a22729fcc0459e9bd -->
- A deployment-posture answer here also carries the full multirepo branching/merge/finishing policy: develop each intent on its own feature branch(es) with matching names across every affected sub-repo and the umbrella repo, never merge to master or deploy to production (human-only), and finish each intent by both pushing to the feature branch(es) and deploying the full system locally to verify. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:practices-discovery:2cccfa904a6bf4263ddfaa3e3936d78fb680ee2dabb16ba9b23989d69c03d1fa -->
- This project uses test-first (TDD) as its standing methodology even though the codebase had zero existing test infrastructure when it was adopted; do not fall back to test-after. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:practices-discovery:bdd78d90350b7144a98b8639005af877d9b11cbcb861eefb30fb2dceaadca58c -->

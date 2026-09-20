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

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
- When a free-text answer to a structured question is already a complete, unambiguous substantive answer (not a request to discuss further), record it verbatim rather than re-prompting the user. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:intent-capture:d9ddd861a20bda0bed4495d91a50a356970674faa64dc18678053057156624fa -->
- When a stage's default question topics reference an upstream artifact type that this scope skips, drop those topics from the questions file rather than asking about an artifact that doesn't exist. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:approval-handoff:745e2f7f09e03d02a54790d170ba5bd17f95a9c42d1603a6127b711316575867 -->
- A `consumes` entry marked required with `consumes_absent` `expected: true` is absent by scope design, not a gap — proceed using only the artifacts that do exist rather than treating it as a missing dependency. (learned 2026-09-20) <!-- cid:260920-quick-filter-presets:approval-handoff:0d0d53bbb41f30686425c531e01637a8f8832a4358b4615a22729fcc0459e9bd -->

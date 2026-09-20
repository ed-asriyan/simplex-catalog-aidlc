**Collaborator:** aidlc-developer-agent

## Contribution

I inspected `simplex-catalog-frontend/src/components/servers/` (`list.svelte`,
`table/index.svelte`, `table/table-header.svelte`, `table/table-row.svelte`,
`fields/line-country.svelte`, `server-modal/index.svelte`) and
`simplex-catalog-frontend/src/store/servers/servers-service.ts`, plus
`package.json`, `tsconfig.json`, and a root/submodule scan for
`.prettierrc*`/`.eslintrc*`. This adds a developer's-eye pass on naming,
boundaries, error handling, file organization, and code style — the areas
`team-practices.md`'s "Code Style" section touches only at the
linter/formatter level today.

### Naming conventions actually in use (not yet affirmed anywhere)

- **Files**: kebab-case throughout for both `.ts` and `.svelte`
  (`servers-service.ts`, `labels-store.ts`, `table-row.svelte`,
  `line-country.svelte`). A folder's entry component is always named
  `index.svelte` (`table/index.svelte`, `server-modal/index.svelte`).
- **TypeScript**: camelCase for variables/functions/methods (`fetch`,
  `addServer`, `countByIdentity`, `generateFilter`, `updateFilter`);
  PascalCase for types/interfaces/classes (`Filter`, `Sort`, `SortField`,
  `ServersService`, `StatusRow`, `SummaryJoinRow`). Component props are
  typed through a local `interface Props { ... }` declared just above the
  `$props()` destructure (see `table/index.svelte` lines 14-25) — a
  consistent, nameable pattern worth affirming explicitly if the team
  wants every Svelte component to follow it.
- **Database-facing fields stay snake_case** (`server_uuid`,
  `last_server_status_uuid`, `info_page_available`, `created_at`) and are
  explicitly mapped to camelCase on the domain `Server` type
  (`infoPageAvailable`, `createdAt`, `lastCheck`) at the service boundary
  (`servers-service.ts` lines 160-179). This boundary-mapping convention is
  real and load-bearing — worth calling out as a candidate "Mandated" rule
  (never leak snake_case DB fields past the service layer) if the team
  confirms it's intentional rather than incidental.
- **Import alias** `@/*` → `src/*` is configured in `tsconfig.json` and used
  consistently for cross-feature imports (`@/store/servers/servers-service`,
  `@/utils`, `@/components/icon.svelte`), while same-directory imports use
  relative paths (`./stats-map.svelte`). Worth affirming as the standing
  convention rather than something a linter enforces (no linter exists).

### Layer / component boundaries

- Feature-based slicing: `src/store/<feature>/` (service + state) is
  cleanly separated from `src/components/<feature>/` (presentation).
  `ServersService` owns all Supabase access, filtering business logic
  (`sortValue`, `compareValues`) and query composition; components only
  read `$derived`/`$state` and call `serversService.fetch(...)` /
  `serversService.addServer(...)`. This existing boundary is worth
  preserving and stating explicitly, since the quick-filter-presets feature
  will add new filter state that should land in the same service/store
  layer rather than in component-local logic.
- Nesting convention: a multi-file subfeature gets its own directory with
  `index.svelte` as the entry point and siblings for internal pieces
  (`table/index.svelte` + `table-header.svelte` + `table-row.svelte`;
  `server-modal/index.svelte` + `stats.svelte` + `stats-plot.svelte`).
  `fields/` breaks this pattern deliberately — it is a flat directory of
  small, reusable leaf renderers shared across the servers feature, not a
  subfeature. This two-tier convention (nested subfeature vs. flat shared
  leaf components) is implicit; worth surfacing to the interview so a new
  "presets" concept is placed consistently (e.g., a `presets/` subfeature
  directory vs. a `fields/`-style shared component).

### Error handling patterns

- Supabase calls in the service layer consistently use `const { data, error
  } = await ...; if (error) throw error;` (three occurrences in
  `servers-service.ts`) — fail-fast, rethrow, no silent swallowing. This is
  a good existing pattern and consistent with the construction-phase
  guardrail against silent failures.
- `addServer` handles Edge Function errors differently: it unwraps
  `requestError.context.json()` and throws a plain `Error` with an
  extracted message (lines 238-248) — a second, ad hoc error shape distinct
  from the plain Supabase `PostgrestError` rethrown elsewhere. Two call
  sites, two different error shapes callers must handle.
- At the UI boundary, `list.svelte`'s `addServerClick` both alerts the user
  *and* rethrows (`catch (e) { alert(e.message); throw e; }`, lines 21-24)
  with nothing upstream catching the rethrow — this reads as leftover/
  inconsistent rather than an intentional pattern, and is the only UI-level
  error handling observed in the files inspected. `@sentry/svelte` is a
  dependency but I found no wiring of these catch blocks to it in the
  inspected files. **Flag for the interview, not asserted as fact**: is
  there a Sentry reporting convention elsewhere in the codebase that
  error-handling for the new preset feature should follow, or should one be
  established now?

### Code style — confirms the draft, adds detail

- Independently confirmed: no `.prettierrc*` / `.eslintrc*` at repo root or
  in `simplex-catalog-frontend/`, and no `eslint`/`prettier` in
  `devDependencies`. `tsconfig.json` exists but only configures
  type-checking (extends `@tsconfig/svelte/tsconfig.json`, `checkJs: true`),
  not formatting or lint rules — it doesn't answer style questions like
  quote style or callback style.
- De facto style observed (useful as "what's already being done by hand,"
  not a substitute for tooling): 4-space indentation, single quotes,
  semicolons, trailing commas in multiline literals.
- One real inconsistency worth surfacing to the interview: callbacks mix
  `const x = async function () {...}` (e.g. `addServerClick`, `changePage`
  in the files read) with arrow functions used everywhere else
  (`$derived(...)` callbacks, inline `onclick`/`onchange` handlers). Nothing
  currently pins this down. If the team adopts a linter now, an arrow-vs-
  function-expression rule would be a natural first win; if not, this is
  low-priority pre-existing debt, not something to block the preset feature
  on.
- `svelte-check` (`npm run check`) is the only static-analysis tool present
  and is not CI-gated — matches `evidence.md` exactly.

### Recommendation for the interview

Beyond the "adopt a linter now vs. defer" question already framed in
`team-practices.md`, I'd add two developer-facing questions:
1. Should the quick-filter-presets feature follow the existing
   `store/<feature>/` (service+state) vs. `components/<feature>/`
   (presentation) split, and should new preset UI live in a new
   `presets/` subfeature directory (à la `table/`, `server-modal/`) or as
   flat additions to the existing `servers` components?
2. Should the new feature's user-facing errors follow the existing
   `alert()`-based pattern, or is this the moment to introduce a
   consistent toast/notification + Sentry-reporting convention, given the
   inconsistency already present in `list.svelte`?

## Positions

AGREE: Testing Posture is correctly left fully open for the interview — package.json confirms no test framework, no test script, and CI runs build-only, so there is nothing to infer from.
AGREE: The Code Style section's core finding (no `.prettierrc*`/`.eslintrc*`, no eslint/prettier dependency, `svelte-check` present but not CI-gated) is independently confirmed byte-for-byte from `package.json` and a fresh scan of repo root and the frontend submodule root.
AGREE: The Deployment section's narrower-than-org-default framing (single GitHub Pages environment, no staging tier, no manual gate, no smoke test) matches what I'd expect from a project this size and is correctly flagged as an open interview question rather than asserted as settled practice.
OBJECT: The draft's "Code Style" section frames the open question purely as "adopt a linter/formatter now or keep deferring," but real naming and file-organization conventions already exist in the code (kebab-case files, `index.svelte` entry-point convention, camelCase↔snake_case boundary mapping, service/component layer separation, a `Props` interface pattern) that aren't captured by any linter and won't be even if one is adopted — the interview should ask the team to affirm these structural/naming conventions explicitly, not just settle the tooling question, so the new preset feature has something concrete to follow from day one.

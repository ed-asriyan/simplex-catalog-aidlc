# Practices Discovery — Evidence

> FINAL. This is a brownfield inspection pass (no reverse-engineering stage ran
> for this scope). It documents what was inspected on disk/in git, what each
> source does and does not tell us about the five practice areas, the
> corrections and additions folded in from the three support agents, and the
> decisions the human made at the interview. The interview outcomes are recorded
> at the end of each section and consolidated in the coverage summary.

## Sources inspected

### 1. Git history and branching

Command: `git -C /home/user/simplex-catalog-aidlc log --oneline -20`

```
8febead Update submodules
a6b9411 Switch to claude
de37093 Update submodule commits for bots, frontend, servers, and supabase
823efa1 Update submodule URLs to use SSH instead of HTTPS
92c6d15 Add Dockerfile and devcontainer configuration for development setup
0cea3db Pull the latest aws aidlc
655cddc Initial commit
```

Command: `git -C /home/user/simplex-catalog-aidlc branch -a`

```
* claude/sharp-lamport-jy3q3f
  master
  remotes/origin/claude/sharp-lamport-jy3q3f
  remotes/origin/master
```

Command: `git -C /home/user/simplex-catalog-aidlc remote -v`

```
origin  https://github.com/ed-asriyan/simplex-catalog-aidlc (fetch)
origin  https://github.com/ed-asriyan/simplex-catalog-aidlc (push)
```

Command: `git -C /home/user/simplex-catalog-aidlc submodule status`

```
-23f3f9e27ddb47e756828cfe9e81d45c6e1f82f0 simplex-catalog-bots-validator
 c55e0ebf8505e8a02450b736a94a187ebfd655b6 simplex-catalog-frontend (heads/master)
-04d7454dc1e5bc592ffba8d238e7e46a5afa1309 simplex-catalog-relays-validator
-06e5618f5ec809218cbe13bead76e2abc7a17cbc simplex-catalog-servers-validator
-1cd61175387517b23e9f18c4aa920904c234e1bd simplex-catalog-supabase
```

`simplex-catalog-frontend` — the submodule relevant to this feature — was
already initialized (leading space, `heads/master` shown), so no
`submodule update --init` was needed for it. The other four submodules show a
leading `-` (uninitialized) and are out of scope for this feature.

**What this tells us:**
- This is a **git-submodule multirepo**: an umbrella repo tracking sub-repos by
  pinned SHA (`Update submodule commits...`).
- The umbrella repo has a short, linear history and its `master` is the only
  long-lived branch; `claude/sharp-lamport-jy3q3f` is the current short-lived
  working branch — consistent with, but not conclusive proof of, short-lived
  feature-branch development at the umbrella level.
- Submodule-pointer commits live in the umbrella repo, which is exactly why a
  cross-repo feature needs an umbrella-repo feature branch too.

**What this did NOT tell us (resolved by the interview):** whether the frontend
submodule itself uses short-lived branches, and what the merge/finishing
convention is. The submodule's own commit graph was not separately inspected.
The **interview settled this authoritatively** (see § 6): a full multirepo
branching/merge/finishing policy now governs — feature branch(es) per intent
with matching names across every affected repo including the umbrella, AI never
merges to master or deploys to production, and each intent ends with both a
commit+push to the feature branch(es) and a local full-system deployment to
verify.

### 2. CI/CD configuration (frontend submodule)

Files read: `/home/user/simplex-catalog-aidlc/simplex-catalog-frontend/.github/workflows/CI.yml`, `build.yml`, `CD.yml`.

**`CI.yml`** — triggers on `pull_request`; calls the reusable `build.yml`
workflow with an `env_file_content` input. No lint step, no test step, no
type-check step — only a build.

**`build.yml`** — reusable `workflow_call`; checks out the repo, writes `.env`
from the supplied content, runs `make prod_build_bundle` (a Dockerized
multi-stage build target), and uploads `dist` as a GitHub Pages artifact.

**`CD.yml`** — triggers on `push` to `master` only; `permissions: contents:
read, pages: write, id-token: write`; `concurrency: group: "pages",
cancel-in-progress: false`. Runs the same `build` job, then a `deploy` job that
runs `actions/deploy-pages@v5` — deploys straight to GitHub Pages with no
manual-approval gate, no separate staging vs. production job, and no smoke-test
step after deploy.

**`Makefile`** (supporting evidence): defines Docker targets `dev`, `staging`
(`app` target, `NODE_ENV=staging`), and `bundle` (`NODE_ENV=production`, the one
CI runs via `prod_build_bundle`). A `staging_serve` target exists for local
staging verification but is not wired into any workflow.

**What this tells us:**
- Deployment cadence: automatic deploy on every push to `master` (single
  environment, GitHub Pages), no separate staging pipeline, no manual approval
  gate. This is a **narrower** picture than `org.md`'s default.
- PR gate: CI runs build-only on every `pull_request` — no lint, no test, no
  `svelte-check` step present today.
- No environment protection rule (`environment:` key) is configured on the
  `deploy` job.

**Interview outcome (§ 6):** the team **keeps** master's GitHub Pages
auto-deploy as-is and adds **no persistent staging environment**. Separately,
the affirmed multirepo policy means the AI never triggers this production deploy
and never merges to master — production deploy occurs only as a consequence of
the human's manual merge. The AI's deploy obligation is a local full-system
verification at intent end. The CI gate is being **strengthened** (tests +
`svelte-check`, see § 3 and § 6), which is a change to `CI.yml`, not to the
deploy cadence.

### 3. Tooling (`package.json`), testing, and dependency automation

File read:
`/home/user/simplex-catalog-aidlc/simplex-catalog-frontend/package.json`.

Scripts: `dev`, `build`, `copy-static`, `preview`, `clean-db`, `stats`,
`check` / `check:watch` (both run `svelte-kit sync && svelte-check`).

`devDependencies` (abridged, tooling-relevant): `svelte` (5.56), `svelte-check`,
`typescript` (6), `vite` (8), `sass`, `@sveltejs/vite-plugin-svelte` (7.2), plus
domain libraries (`d3`, `@supabase/supabase-js`, `nanostores`, `uikit`,
`moment`, `i18n-iso-countries`, `topojson-client`, `@sentry/svelte`,
`@castlenine/svelte-qrcode`, `@mateothegreat/svelte5-router`). This is a
Vite-native build, not SvelteKit's full-stack framework (no `@sveltejs/kit`
despite the `svelte-kit sync` call in `check`, a scaffolding vestige used only
for TS sync).

**No test framework dependency present** — no `vitest`, `jest`,
`@testing-library/*`, `playwright`, `@testing-library/svelte`, or coverage tool,
and no `test` script. Independently confirmed by the quality agent, which also
searched the whole `src/` tree for `*.test.*`/`*.spec.*` and found **zero
files**. There is no existing test suite of any kind to be compatible with.

**No linter or formatter config found** — no `.prettierrc*` / `.eslintrc*` at
repo root or the submodule root, and neither `eslint` nor `prettier` in
`devDependencies` (independently confirmed by the developer agent). The only
static-analysis-adjacent tool is `svelte-check` (`npm run check`), which is not
CI-wired. `tsconfig.json` configures type-checking only (extends
`@tsconfig/svelte/tsconfig.json`, `checkJs: true`); it does not answer style
questions.

**Dependency automation DOES exist (correction).** Contrary to the earlier
draft of this evidence, `simplex-catalog-frontend/.github/dependabot.yml`
**exists** and is configured for four ecosystems — `npm`, `docker`,
`github-actions`, `devcontainers` — each on a weekly schedule,
`open-pull-requests-limit: 1`, ignoring patch-only bumps (surfaced and
confirmed by the devsecops agent). It is important to characterize this
precisely: **Dependabot here is update automation, not a CI-blocking
vulnerability gate.** Nothing in `CI.yml`/`build.yml` runs `npm audit` (or
equivalent) to fail the build on a Critical/High CVE, so a vulnerable
dependency can still be merged between Dependabot's weekly PR and review.
Whether GitHub's Dependabot *security alerts* and secret scanning are enabled
is a repo/org setting not visible from checked-out files and was not confirmed.

**What this tells us:**
- Code style: there is no linter/formatter config to defer to yet;
  `svelte-check` exists but is a manual, non-CI-gated check.
- Testing: a **green field** for the testing-posture decision — nothing to be
  compatible with, so methodology and tooling had to come from the human.
- Dependency hygiene: automated update PRs exist (Dependabot), but no
  CI-blocking vulnerability gate does.

**Interview outcome (§ 6):** the team **adopts Vitest** (with
`@vitest/coverage-v8` and `@testing-library/svelte`), **adopts ESLint +
Prettier now**, and wires **tests + `svelte-check`** into CI as required merge
gates. A CI-blocking `npm audit`/SAST gate was noted by devsecops as a
pre-existing gap but was not adopted at this stage (the feature adds no new
security-sensitive surface — see § 5).

### 4. Code conventions actually in use (developer inspection)

The developer agent inspected `simplex-catalog-frontend/src/components/servers/`
(`list.svelte`, `table/index.svelte`, `table/table-header.svelte`,
`table/table-row.svelte`, `fields/line-country.svelte`,
`server-modal/index.svelte`) and
`simplex-catalog-frontend/src/store/servers/servers-service.ts`, plus
`tsconfig.json`. Real, previously-unwritten conventions surfaced:

- **Filenames**: kebab-case for `.ts` and `.svelte`. A folder's entry component
  is always `index.svelte`.
- **TypeScript**: camelCase for variables/functions/methods; PascalCase for
  types/interfaces/classes. Component props typed via a local `interface Props`
  declared just above the `$props()` destructure.
- **Boundary mapping**: database-facing fields stay snake_case (`server_uuid`,
  `info_page_available`, `created_at`) and are mapped to camelCase on the domain
  type at the service boundary (`servers-service.ts`) — DB snake_case does not
  leak past the service.
- **Import alias**: `@/*` → `src/*` for cross-feature imports; relative paths
  for same-directory imports.
- **Layering**: `src/store/<feature>/` (service + state) is cleanly separated
  from `src/components/<feature>/` (presentation); services own Supabase access
  and filtering logic, components only read `$derived`/`$state` and call service
  methods.
- **Two-tier nesting**: a multi-file subfeature gets its own directory with
  `index.svelte` + sibling internal pieces (`table/`, `server-modal/`);
  `fields/` is a flat directory of small reusable leaf renderers, deliberately
  not a subfeature.
- **Error handling** (observed, informational): service-layer Supabase calls use
  fail-fast `if (error) throw error;`; `addServer` uses a second ad hoc error
  shape; `list.svelte`'s `addServerClick` both `alert()`s and rethrows with
  nothing catching the rethrow (reads as inconsistent). `@sentry/svelte` is a
  dependency but no wiring of these catch blocks to it was found in the
  inspected files. Not blocking; flagged for functional/domain design.
- **De facto style** (not a substitute for tooling): 4-space indentation, single
  quotes, semicolons, trailing commas in multiline literals; callbacks mix
  `async function` expressions with arrow functions inconsistently.

**Interview outcome (§ 6):** the structural/naming conventions above
(kebab-case files, `index.svelte` entry, `interface Props`, service-layer
snake_case↔camelCase mapping, store/component layering, two-tier nesting, `@/*`
alias) are **affirmed as team practice**, on top of adopting ESLint + Prettier.

### 5. Security posture (devsecops inspection)

The devsecops agent assessed the feature and the pipeline:

- **This feature introduces no new security-sensitive surface.** Quick filter
  presets plus localStorage-backed custom filters are client-side only: no new
  backend endpoint, no new auth/authorization path, no new secret or credential.
  Persisted data (a user's own filter selections) is non-sensitive, never
  transmitted, and stays in the browser's `localStorage` origin — consistent
  with existing localStorage use in `bots/list.svelte`, `relays/list.svelte`,
  `servers/list.svelte`, and `store/servers/labels-store.ts`.
- **Two non-blocking implementation notes to carry into design:**
  1. Treat data read back from `localStorage` as **untrusted input at the
     boundary** (validate shape/type before use) — it can be edited via devtools
     or another extension.
  2. Svelte auto-escapes template output and no `{@html}` usage was found in
     `src/`; the new preset-name UI should **avoid introducing any `{@html}` or
     `innerHTML` sink** for user-typed preset names, keeping stored/DOM XSS risk
     low.
- **Pre-existing pipeline gaps (not introduced or worsened by this feature):**
  no SAST config, no in-repo secret-scanning tooling, no `npm audit`/CI
  dependency-vulnerability gate. Dependabot (§ 3) provides update PRs but not a
  blocking gate. `build.yml` inlines every `VITE_*` variable into the public
  bundle (by design for the Supabase anon key, protected by RLS) — worth the
  team confirming no future `VITE_*` var is ever a true secret.

None of these block or resize this feature's scope.

### 6. Interview decisions (authoritative)

Recorded from the filled `[Answer]` tags in `practices-discovery-questions.md`:

- **Q1 Way of Working** — short-lived feature branches, squash-merged, refined
  by the fuller multirepo policy (Q7/Q7-follow-up): feature branch(es) per
  intent with matching names across every affected repo including the umbrella;
  AI never merges to master; each intent ends with both commit+push to the
  feature branch(es) and a local full-system deployment to verify.
- **Q2 Walking Skeleton** — **always** build a thin end-to-end slice first,
  regardless of feature size.
- **Q3 Test framework** — **Vitest** (fits the Vite/Svelte 5 setup).
- **Q4 Methodology** — **test-first / TDD**.
- **Q5 Coverage + CI** — **80% line coverage on new/changed code only**, and CI
  **fails the PR if tests don't pass**.
- **Q6 `svelte-check`** — **yes**, add it as a required CI check.
- **Q7 Deployment** — master keeps auto-deploying to GitHub Pages as today; **no
  persistent staging environment**; plus the multirepo branching/merge/local-
  deploy policy above (AI never merges to master, never deploys to production;
  human merges manually at their discretion; branches are fresh per intent).
- **Q8 Code Style** — **adopt ESLint + Prettier now** and wire into CI; affirm
  the existing naming/layering conventions from § 4.

### 7. Space defaults already in force

- `aidlc/spaces/default/memory/org.md` — supplied the framework defaults used as
  starting proposals before the interview (trunk-based dev, squash-merge,
  deploy-on-merge-to-staging with a manual prod gate, defer-to-project-config
  code style). Where the interview affirmed something narrower or more specific
  (single-environment deploy, multirepo policy, TDD, adopt-linter-now), the
  affirmed team practice governs.
- `aidlc/spaces/default/memory/project.md` — carries 3 learned `## Corrections`
  entries from earlier stages (`intent-capture`, `approval-handoff`), none of
  which touch these five practice areas. This is the **first**
  practices-discovery run for this project; `team.md` was template-empty across
  all five headings before it.

## Coverage summary by practice area

| Area | Evidence strength | Interview outcome |
|---|---|---|
| Way of Working (branching/merge) | Partial from repo; resolved by interview | Multirepo policy affirmed: feature branch(es) per intent, matching names across every affected repo incl. umbrella; AI never merges to master; intent ends with commit+push AND local full-system deploy; squash-merge; fresh branches per intent. |
| Walking Skeleton | No direct repo evidence | Always build a thin end-to-end slice first, regardless of feature size. |
| Testing Posture | Green field (no framework/test/CI step) | TDD (test-first); Vitest + @vitest/coverage-v8 + @testing-library/svelte; 80% line coverage on new/changed code only; CI fails PR on test failure; `svelte-check` added as a required CI gate. |
| Deployment | Well-evidenced (single GitHub Pages env, no gate/smoke test) | Keep master → GitHub Pages auto-deploy as-is; no persistent staging; AI never deploys to production or merges to master; AI's deploy duty is local full-system verification at intent end. |
| Code Style | Absent tooling; real conventions in code | Adopt ESLint + Prettier now, wired into CI; affirm existing conventions (kebab-case files, `index.svelte` entry, `interface Props`, service-layer snake_case↔camelCase mapping, store/component layering, two-tier nesting, `@/*` alias). |
| Dependency automation | Corrected: Dependabot exists | `dependabot.yml` (npm/docker/github-actions/devcontainers, weekly) exists — update automation, not a CI-blocking vuln gate. No `npm audit`/SAST gate adopted this stage; noted as a pre-existing gap. |
| Feature security | Inspected | No new security-sensitive surface (client-side, localStorage, no new auth/secrets). Treat localStorage as untrusted at the boundary; avoid `{@html}` sinks for user-typed preset names. |

## Residual uncertainty

- Whether GitHub Dependabot **security alerts** and **secret scanning** are
  enabled at the repo/org level is not visible from checked-out files and was
  not confirmed — recommend confirming in GitHub settings.
- The frontend submodule's own historical branch/merge convention was not
  separately inspected from its commit graph; the affirmed multirepo policy
  supersedes the need to infer it, but it remains an unverified historical fact.
- A CI-blocking dependency-vulnerability gate (`npm audit --audit-level=high`)
  and a SAST pass (`eslint-plugin-security`) were surfaced as pre-existing gaps
  but not adopted this stage; they remain open future decisions.
- The existing UI error-handling inconsistency (`alert()` + rethrow, ad hoc
  error shapes, unwired `@sentry/svelte`) is flagged for functional/domain
  design, not resolved here.

# Practices Discovery — Evidence

> DRAFT. This is a brownfield inspection pass (no reverse-engineering stage
> ran for this scope). It documents exactly what was inspected on disk/in
> git and what each source does and does not tell us about the five
> practice areas the human interview at this stage needs to affirm.

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
`submodule update --init` was needed for it. The other four submodules
show a leading `-` (uninitialized) and are out of scope for this feature.

**What this tells us:**
- The umbrella repo has a short, linear history (6-7 commits) — a single
  operator/bootstrap history, not yet showing a repeated feature-branch
  cadence at the top level.
- The umbrella repo's own `master` branch is the only long-lived branch;
  the `claude/sharp-lamport-jy3q3f` branch is the current working branch
  (short-lived, single feature branch pattern — consistent with, but not
  conclusive proof of, trunk-based development at the umbrella level).
- The umbrella repo tracks submodule commits by pinned SHA (`Update
  submodule commits...`), which is a common pattern for a poly-repo/
  submodule setup but says nothing about the frontend submodule's own
  internal branching or PR conventions, since that history lives in the
  submodule's own repo, not surfaced by these commands.

**What this does NOT tell us:** Whether the *frontend* submodule itself
uses short-lived branches, what its merge convention is (squash vs.
merge-commit vs. rebase), or how long branches typically live there — its
own commit graph was not separately inspected. `org.md`'s trunk-based /
squash-merge default is therefore a plausible default to propose, not a
confirmed observation.

### 2. CI/CD configuration (frontend submodule)

Files read: `/home/user/simplex-catalog-aidlc/simplex-catalog-frontend/.github/workflows/CI.yml`, `build.yml`, `CD.yml`.

**`CI.yml`** — triggers on `pull_request`; calls the reusable `build.yml`
workflow with an `env_file_content` input. No lint step, no test step —
only a build.

**`build.yml`** — reusable `workflow_call`; checks out the repo, writes
`.env` from the supplied content, runs `make prod_build_bundle` (a
Dockerized multi-stage build target — see `Makefile` below), and uploads
`dist` as a GitHub Pages artifact.

**`CD.yml`** — triggers on `push` to `master` only; `permissions:
contents: read, pages: write, id-token: write`; `concurrency: group:
"pages", cancel-in-progress: false`. Runs the same `build` job, then a
`deploy` job that runs `actions/deploy-pages@v5` — deploys straight to
GitHub Pages with no manual-approval gate, no separate staging vs.
production job, and no smoke-test step after deploy.

**`Makefile`** (supporting evidence for the build step): defines Docker
targets `dev`, `staging` (`app` target, `NODE_ENV=staging`), and `bundle`
(`NODE_ENV=production`, the one CI actually runs via
`prod_build_bundle`). A `staging_serve` target exists for local staging
verification but is not wired into any workflow.

**What this tells us:**
- Deployment cadence: automatic deploy on every push to `master` (single
  environment, GitHub Pages), no separate staging deployment pipeline and
  no manual approval gate observed in the workflow files. This is a
  **narrower** picture than `org.md`'s default ("deploy on merge to
  staging; production gates on manual approval") — there is only one
  deploy target here (GitHub Pages) and it fires unconditionally on
  merge to `master`.
- PR gate: CI runs build-only on every `pull_request` — no lint, no test,
  no type-check (`svelte-check`) step present in `CI.yml` today.
- No `deployment protection rules` / environment gate is configured in
  the YAML (would show as an `environment:` key on the `deploy` job) —
  none present.

**What this does NOT tell us:** Whether the team wants to keep
"deploy-on-merge-to-Pages-with-no-gate" as affirmed practice, or whether
this is simply what a small/early project has today and the human wants
something closer to the org default (staging + manual prod gate) once
more environments exist. This is a **Deployment** question for the
interview: confirm whether the current single-environment/no-gate setup
is intentional policy or an artifact of project youth.

### 3. Tooling (`package.json`)

File read:
`/home/user/simplex-catalog-aidlc/simplex-catalog-frontend/package.json`.

Scripts: `dev`, `build`, `copy-static`, `preview`, `clean-db`, `stats`,
`check` / `check:watch` (both run `svelte-kit sync && svelte-check`).

`devDependencies` (abridged, tooling-relevant): `svelte`, `svelte-check`,
`typescript`, `vite`, `sass`, `@sveltejs/vite-plugin-svelte`, plus
domain libraries (`d3`, `@supabase/supabase-js`, `nanostores`, `uikit`,
`moment`, `i18n-iso-countries`, `topojson-client`, `@sentry/svelte`,
`@castlenine/svelte-qrcode`, `@mateothegreat/svelte5-router`).

**No test framework dependency present** — no `vitest`, `jest`,
`@testing-library/*`, `playwright`, or `@testing-library/svelte` in
`devDependencies`, and no `test` script. Confirmed also that CI (`CI.yml`
→ `build.yml`) runs no test step.

**No linter or formatter config found**: a scan for `.prettierrc*` /
`.eslintrc*` at the submodule root returned nothing, and neither
`eslint` nor `prettier` appear in `devDependencies`. The only
static-analysis-adjacent tool present is `svelte-check` (TypeScript/
Svelte type-checking via `npm run check`), which is not wired into CI
either.

**What this tells us:**
- Code style: there is no linter/formatter config to defer to yet at the
  frontend-submodule level (org.md's "defer to project config" default
  has no project config to find here). `svelte-check` exists as a
  type-check script but is a manual, not CI-gated, check today.
- Testing: there is **no existing test framework or test to be
  compatible with** — this is a green field for the testing-posture
  decision, not a case of "infer from what's there."

**What this does NOT tell us:** Which test framework/methodology the
team wants going forward (Vitest is the natural fit for a Vite/Svelte
stack, but no evidence in the repo confirms that preference — it would be
a proposal, not an observation). This is squarely what this stage's
human interview needs to resolve, per the task brief; the draft
`team-practices.md` deliberately leaves this open rather than asserting a
choice.

### 4. Space defaults already in force

- `aidlc/spaces/default/memory/org.md` — supplies the framework defaults
  the draft below treats as *suggested* starting points wherever the
  repo evidence above doesn't already show something more specific:
  trunk-based dev with short-lived branches, squash-merge Bolts,
  deploy-on-merge-to-staging with a manual production gate, and
  "defer to project linter/formatter config" for code style.
- `aidlc/spaces/default/memory/project.md` — already carries 3 learned
  `## Corrections` entries from earlier stages in this workflow
  (`intent-capture` and `approval-handoff`), none of which touch Way of
  Working / Testing / Deployment / Code Style; all other `project.md`
  sections (`Way of Working`, `Testing Posture`, `Deployment`, `Code
  Style`, `Tech Stack`, `Decided`, `Scope Overrides`, `Forbidden`,
  `Mandated`) are template-empty. This is the **first**
  practices-discovery run for this project — `team.md` is still
  template-empty across all five headings.

## Coverage summary by practice area

| Area | Evidence strength | Notes |
|---|---|---|
| Way of Working (branching/merge) | Partial | Umbrella repo shows one short-lived working branch off `master`; frontend submodule's own branch/merge history was not separately inspected. org.md trunk-based/squash default is a reasonable starting proposal, not a confirmed fact. |
| Walking Skeleton | No direct evidence | Nothing in git history or CI/CD indicates whether a skeleton-first practice has ever been used on this project; this is scope-file driven (`skeleton: on/off`) rather than something to infer from repo state. |
| Testing Posture | **Needs interview** | No test framework, no test script, no CI test step. This is exactly the gap the interview must close — methodology and tooling choice, not something to infer from absence. |
| Deployment | Well-evidenced, but narrower than org default | CI.yml/build.yml/CD.yml give a concrete, current picture: build-only PR gate, auto-deploy to GitHub Pages on push to `master`, one environment, no manual approval gate, no smoke test. Needs the interview to confirm whether this is affirmed policy or should move toward the org default (staging + prod approval) as more environments appear. |
| Code Style | Evidenced as absent | No linter/formatter config exists at the frontend-submodule root; `svelte-check` exists but is not CI-gated. org.md's "defer to project config" default has nothing to defer to yet — the interview should confirm whether to adopt a linter/formatter now or continue deferring. |

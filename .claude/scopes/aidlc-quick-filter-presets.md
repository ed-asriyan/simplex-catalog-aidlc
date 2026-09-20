---
name: quick-filter-presets
depth: Standard
keywords: []
description: Hardcoded quick-filter preset buttons plus create/save/edit/persist custom filters in localStorage, for the servers page
skeleton: off
review_cap: advisory
change_control: relaxed
---

# quick-filter-presets scope

Composed by the adaptive-workflows composer for a single request: "Right
now, users have to manually enter filters every time on the servers page
... provide a set of hardcoded presets that immediately apply that
preset's view on click. On top of that, users should be able to create,
save, edit, and persist custom filters in localStorage. We need to design
this flow."

This is a single-package, low-coupling frontend feature on an existing,
already-mapped page (`simplex-catalog-frontend`), with no test framework in
place yet and moderate open questions about the exact preset list and the
custom-filter CRUD UX. Depth is Standard because real Inception-phase
design work runs (requirements-analysis, user-stories, refined-mockups,
practices-discovery) even though the stage count was deliberately pulled
lean via composer folds — the composite ARS scored 44 ("Standard" band)
but the mechanical stage count was cut roughly in half by folding
overlapping ideation/inception stages and NFR/observability/performance
stages that don't apply to a client-side filter feature.

Change Control defaults to relaxed: an input that changes after approval
is recorded and announced in one line rather than reopening the approval,
matching a solo, low-risk, fully-reversible client-side change.

## ARS at composition

| Component | Symbol | Score | Band |
|-----------|--------|-------|------|
| Intent Ambiguity | IAE | 0.40 | MED |
| Codebase Structural Uncertainty | CSU | 0.28 | LOW |
| Verification Entropy | VE | 0.75 | HIGH |
| Risk / Blast Radius | R | 0.30 | MED |
| Unresolved Assumptions | UA | 0.45 | MED |
| **Composite ARS (advisory)** | - | **44 / 100** | **Standard** |

Method: fallback (no CodeKB tools available in the composing session;
scored from a direct read of the initialized `simplex-catalog-frontend`
submodule).

## Why these stages, why skip those

**Runs:** intent-capture (pins down which presets and the save/edit UX),
approval-handoff (ideation gate), practices-discovery (the frontend has no
test framework at all — pick one before writing testable code),
requirements-analysis (sole spec stage; domain/functional design are
skipped), user-stories (human-requested: two distinct, independently
testable flows — click-a-preset vs. create/edit/save-a-custom-filter — get
their own Given/When/Then acceptance criteria per the inception phase
rules), refined-mockups (UX-heavy, user-facing change; absorbs
rough-mockups' role directly since the UI already exists), code-generation
and build-and-test (the spine), and ci-pipeline (the existing CI workflow
only builds — it needs a test-execution step for the new tests).

**Skips, folded:** market-research (known, existing product; nothing to
research), feasibility (a standard, well-understood browser pattern — no
viability question), scope-definition (already tightly scoped to one
page; requirements-analysis pins the exact boundary), team-formation (no
multi-team coordination), rough-mockups (folded into refined-mockups —
brownfield redesign of an already-mapped page).

**Skips, below threshold:** reverse-engineering (CSU=0.28, LOW — the
affected code was located and understood directly), domain-design,
units-generation, contract-design, delivery-planning, functional-design
(single unit, low structural uncertainty — no component-model or
decomposition work needed), nfr-requirements and nfr-design (the
verification-entropy score is driven by missing test tooling, not by a
real performance/security/compliance concern — this feature introduces no
new NFR), infrastructure-design, deployment-pipeline,
environment-provisioning, deployment-execution (R=0.30 — the existing
deploy-on-merge pipeline needs no change), observability-setup (Sentry is
already integrated — no new service), incident-response (no new
operational surface), performance-validation (client-side filtering and
localStorage reads/writes are negligible-cost and not an explicit NFR),
and feedback-optimization (no post-launch iteration loop was requested).

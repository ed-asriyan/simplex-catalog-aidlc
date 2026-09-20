# Intent Statement — Quick Filter Presets

## Problem Statement

Users of the servers page have to manually re-enter and rebuild their filter criteria every time they want to switch between different views of the servers list, which is slow and repetitive [desc] [Q1]. There is currently only one ad-hoc filter, mirrored to a single "last used filter" `localStorage` key, with no way to save or instantly switch between multiple named views [desc].

## Target Customer

Any visitor/user of the public servers catalog page who filters servers repeatedly [Q2]. This is a public-facing catalog, so "customer" is not limited to internal staff.

## Success Metrics

Success is primarily the qualitative experience of applying a saved view in one click instead of rebuilding filters manually [Q3]. Usage analytics also matter: the requester wants to be able to measure, after release, which filters/presets are used more often than others [Q3]. No specific numeric target was set for this measurement at this stage.

## Initiative Trigger

The initiative is driven by dogfooding — the requester personally needs to switch between multiple views of the servers list (examples given: "all active clearnet," "only tor," "90+ uptime" — explicitly non-exhaustive examples, not a final list) [Q4]. The request is explicitly modeled on Jira's Quick Filters as a reference pattern [desc]. The requester also asked that the workflow explore what other preset filters would be useful, and confirmed that users should be able to create their own custom filters in addition to the hardcoded presets [Q4] [desc].

## Initial Scope Signal

- **Workflow-selected scope**: `quick-filter-presets` (custom-tailored plan) [scope]
- **User-confirmed product boundary**: Confirmed as matching — a 12-step plan covering intent capture, approval & handoff, practices discovery (picking a test framework, since the frontend currently has none), requirements analysis, user stories, refined mockups, code generation, build & test, and CI pipeline wiring, while skipping market research, feasibility study, formal team formation, and deep architecture/domain-design work [Q8].

## Assumptions & Open Questions

- The exact set of hardcoded quick-filter presets (beyond the illustrative examples given — active clearnet, tor only, 90+ uptime) is not yet defined and needs to be worked out, informed by what filterable attributes the servers catalog actually exposes. [assumption]
- Whether there is a limit on the number of custom filters a user can save, and whether custom filters need names distinct from presets, is not yet defined. [assumption]
- The specific analytics mechanism for tracking per-filter/per-preset usage (e.g., via the existing Sentry integration or another tool) is not yet defined. [assumption]

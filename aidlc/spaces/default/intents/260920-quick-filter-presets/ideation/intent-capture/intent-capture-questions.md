# Intent Capture & Framing — Questions

## Sources

- [desc] Initial description: "Right now, users have to manually enter filters every time on the servers page. This is inconvenient because whenever they tweak something, they have to switch between filters. It would be much better to have quick filter preset buttons—similar to Jira's Quick Filters—so users can click one to instantly apply a predefined filter, or use their own ad-hoc, unsaved filters. We should provide a set of hardcoded presets that immediately apply that preset's view on click. On top of that, users should be able to create, save, edit, and persist custom filters in localStorage. We need to design this flow."
- [scope] Workflow-selected scope: `quick-filter-presets`.

## Q1. What business problem are we solving?

Today, users of the servers page have to manually rebuild their filter criteria every time they want to view a different slice of the data, and switching between different views means re-entering filters from scratch each time.

- A. Users must manually re-enter/rebuild filter criteria every time they want to switch between different views of the servers list, which is slow and repetitive [desc]
- B. The servers page filtering is not usable at all today (a more severe problem than repetitive re-entry)
- C. A different business problem than either of the above
- D. Not yet defined
- X. Other (please specify)

[Answer]: A. Users must manually re-enter/rebuild filter criteria every time they want to switch between different views of the servers list, which is slow and repetitive

## Q2. Who is the customer for this feature (internal/external)? What pain are they experiencing?

- A. Any visitor/user of the servers catalog page who filters servers repeatedly — this is a public-facing catalog, so "customer" means any site visitor using the filters [desc]
- B. Internal team members only (admins/staff who maintain the catalog)
- C. Both external visitors and internal staff, with the same pain
- D. Not yet defined
- X. Other (please specify)

[Answer]: A. Any visitor/user of the servers catalog page who filters servers repeatedly — this is a public-facing catalog, so "customer" means any site visitor using the filters

## Q3. What does success look like for this feature? What metrics matter?

- A. Success is qualitative: users can apply a saved view in one click instead of rebuilding filters manually — no specific numeric metric tracked
- B. Success is measurable: increased filter-panel usage / reduced average time spent adjusting filters, tracked via the existing Sentry error/usage tooling already in the frontend
- C. Success is measured some other way
- D. Not yet defined
- X. Other (please specify)

[Answer]: X. Other (please specify) — Success is the one-click saved-view experience (as in A), but metrics do matter: usage analytics must be in place so that after release it's possible to measure which filters/presets are used more often than others.

## Q4. What is the trigger for this initiative (market pressure, tech debt, regulation, opportunity)?

- A. Direct observation/feedback that the current single ad-hoc filter (today mirrored to one "last used filter" localStorage key) is repetitive to rebuild on every visit [desc]
- B. A desire for feature parity with familiar tools like Jira's Quick Filters, which the request explicitly references as the model [desc]
- C. Both A and B together
- D. Not yet defined
- X. Other (please specify)

[Answer]: X. Other (please specify) — Dogfooding: the requester personally needs to switch between multiple views of the servers list (e.g. "all active clearnet", "only tor", "90+ uptime" — given as examples, not an exhaustive list) and wants the workflow to explore what other preset filters would be useful, in addition to letting users create their own custom filters.

## Q5. Who are the key stakeholders and what does each care about?

- A. You (the person requesting this work), as the product owner — cares that the feature ships correctly and matches the described flow
- B. You, plus the end users of the servers catalog, who care about ease of use
- C. There are additional stakeholders beyond A/B (e.g. a design or ops team)
- D. Not yet defined
- X. Other (please specify)

[Answer]: B. You, plus the end users of the servers catalog, who care about ease of use

## Q6. Who decides scope or priority, and who influences those decisions?

- A. You are the sole decision-maker for scope and priority on this piece of work
- B. You decide, but with input from a small team
- C. Someone else entirely decides
- D. Not yet defined
- X. Other (please specify)

[Answer]: A. You are the sole decision-maker for scope and priority on this piece of work

## Q7. Are there communication requirements or a reporting cadence for this work?

- A. None — ship once complete and reviewed through the normal pull-request flow
- B. A demo or announcement to stakeholders is needed before or after shipping
- C. Some other communication requirement applies
- D. Not applicable
- X. Other (please specify)

[Answer]: X. Other (please specify) — There is a community chat of about 700 users; the requester will post/announce the feature there once it ships.

## Q8. Scope confirmation

This workflow started with a custom-tailored plan (`quick-filter-presets`) rather than a stock scope: intent capture → approval & handoff → practices discovery (the frontend currently has no test framework, so one gets picked here) → requirements analysis → user stories → refined mockups → code generation → build & test → CI pipeline wiring — 12 steps in total, skipping market research, feasibility study, formal team formation, and deep architecture/domain-design work as unnecessary for a single-package, low-risk UI feature.

Does that match your intended product boundary for this work, or is there a different boundary you have in mind (e.g. broader — should custom filters sync across devices, or narrower — presets only, no custom filters yet)?

- A. Yes, that scope and boundary match what I want built
- B. No, I want a different boundary — I'll specify what should change
- X. Other (please specify)

[Answer]: A. Yes, that scope and boundary match what I want built

## Assumption Confirmation

The following assumptions were preserved in `intent-statement.md` because they are useful context but were not directly confirmed by an answer above:

- The exact set of hardcoded quick-filter presets (beyond the illustrative examples given — active clearnet, tor only, 90+ uptime) is not yet defined and needs to be worked out, informed by what filterable attributes the servers catalog actually exposes.
- Whether there is a limit on the number of custom filters a user can save, and whether custom filters need names distinct from presets, is not yet defined.
- The specific analytics mechanism for tracking per-filter/per-preset usage (e.g., via the existing Sentry integration or another tool) is not yet defined.

- A. Accept assumptions
- B. Convert to follow-up questions

[Answer]: A. Accept assumptions

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct

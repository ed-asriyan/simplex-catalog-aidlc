# Personas

## Sources

- Requirements `../requirements-analysis/requirements.md` (`## Actors`)
- Intent statement `../../ideation/intent-capture/intent-statement.md`
- Plan answers `user-stories-questions.md` (Q1 = one persona)

## Assumptions & Open Questions

None.

## P1 — Catalog Visitor (primary, only persona)

- **Role**: Anyone browsing the public SimpleX servers catalog. Unauthenticated;
  all state is per-browser. No accounts, no roles, no server-side profile.
- **Goals**:
  - See the specific slice of servers they care about right now (all online,
    online clearnet, online tor, recently added, high uptime) without hand-building
    a filter each time.
  - Switch quickly between several recurring "views" during a single session.
  - Save and re-apply their own recurring filters so a personal view survives
    reloads and is one click away.
  - Share or bookmark a specific view via its URL.
- **Pain points (today)**:
  - The filter must be re-entered manually every time; tweaking one facet means
    fiddling with multiple controls.
  - There is exactly one "last used" filter in localStorage — no way to keep
    several named views.
  - Common views ("active clearnet", "tor only", "90%+ uptime") are not one click.
- **Context / behaviour spectrum** (captured within this one persona, not split
  into separate personas per Q1):
  - *Occasional browser*: lands on the page, clicks a preset, scans results.
    Rarely saves a custom filter.
  - *Power / dogfooding user* (e.g. the maintainer): switches views constantly
    within a session — sometimes all active clearnet, sometimes only tor,
    sometimes 90%+ uptime — and wants to save their own recurring filters.
    This is the primary driver behind custom filters.
- **Priority ranking**: Sole persona. Both the preset flow (serves the whole
  spectrum) and the custom-filter flow (serves the power/dogfooding end) must be
  satisfied for this persona.

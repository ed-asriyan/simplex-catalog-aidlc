# Accessibility Checklist — Quick-Filter Presets & Custom Filters

Target: **WCAG 2.1 AA** (Q4=A). Satisfies NFR4 and the a11y acceptance criteria
(AC5.1.7 keyboard, AC5.1.8 management controls, AC5.1.9 AT-exposed selected state).

## Sources

- Requirements `../requirements-analysis/requirements.md` (NFR4)
- User stories `../user-stories/stories.md` (AC5.1.7–AC5.1.9, AC2.3.4)
- Interaction spec `interaction-spec.md`

## Assumptions & Open Questions

None.

## Checklist

### Keyboard operability (AC5.1.7, AC5.1.8)

- [ ] Every preset chip is reachable by Tab and activatable with Enter/Space (native `<button>`).
- [ ] Each custom chip body is Tab-reachable and Enter/Space-activatable.
- [ ] The ⋯ trigger is a separate Tab stop and Enter/Space opens its menu.
- [ ] The ⋯ menu items (Rename… / Update to current view / Delete) are Tab/arrow reachable and Enter/Space-activatable.
- [ ] Esc closes the ⋯ menu and returns focus to the ⋯ trigger.
- [ ] The "Save current" control is Tab-reachable and keyboard-activatable.
- [ ] Focus order is left→right along the bar, matching visual order.
- [ ] No keyboard trap in any menu or native prompt/confirm.

### Screen reader / ARIA (AC5.1.9)

- [ ] Bar container is `role="group"` with `aria-label="Quick filters"`.
- [ ] Active chip exposes `aria-pressed="true"`; inactive chips `aria-pressed="false"`.
- [ ] Exactly one chip is `aria-pressed="true"` at a time (or none when ad-hoc) — matches the visible selected state (AC5.1.4).
- [ ] ⋯ trigger has an `aria-label` naming the filter (e.g. "Manage My clearnet 90+").
- [ ] Menu items have descriptive accessible names.
- [ ] Native `prompt()`/`confirm()` dialogs are inherently screen-reader accessible (browser-native).

### Visible focus & contrast

- [ ] Visible focus indicator on every chip, the ⋯ trigger, menu items, and Save control (UIkit default focus ring; do not remove outline).
- [ ] Text/background contrast ≥ 4.5:1 for chip labels; UI component contrast ≥ 3:1 (UIkit primary/default already AA).
- [ ] Selected state is not conveyed by color alone — `aria-pressed` + the primary style's weight/shape difference provide a non-color cue; consider a check glyph on the active chip if color contrast of the state is borderline.

### Target size & responsive

- [ ] Chips and controls are ≥ 32px tall touch targets; adequate spacing when wrapped (AA best practice).
- [ ] Bar reflows without horizontal scrolling at 320px width.

### States & feedback

- [ ] Empty-custom state (no saved filters) still shows presets + Save control; nothing is announced as an error (AC2.1.6).
- [ ] Storage-unavailable/malformed degrades to presets only, no error surfaced (AC3.1.3, NFR5).
- [ ] Empty result set for a preset shows the existing count block, not an error, and the chip stays selected (AC1.1.5).
- [ ] Delete requires a confirm step (AC2.3.4); the confirm text names the filter.

### Verification method

- [ ] Manual keyboard-only pass through apply / save / rename / update / delete / modify-then-deselect.
- [ ] Screen-reader spot check (VoiceOver or NVDA) for pressed state + menu labels.
- [ ] `@testing-library/svelte` tests assert `aria-pressed` toggles and that ⋯-menu actions do not apply the filter (AC2.3.3) — wired in Construction per the TDD posture.

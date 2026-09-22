# [a11y] Invalid ARIA and unnamed controls from the archived Shoelace beta

**Labels:** accessibility, dependencies

## Summary
Several accessibility defects originate inside Shoelace `2.0.0-beta.71`, not
medblocks source, and cannot be fixed cleanly while on that beta. Shoelace is now
**archived / end-of-life** (its repo description reads "Shoelace is now Web
Awesome"; final release 2.20.1, March 2025). This issue tracks the Shoelace-rooted
findings and the strategic dependency decision.

## Findings (verified)
1. **Invalid ARIA — `aria-checked` on `role="menuitem"`** (WCAG 4.1.2). Shoelace's
   `sl-menu-item` self-assigns `role="menuitem"` + `aria-checked`, an invalid
   combination and the wrong pattern for choosable results. Affects the option
   lists in **mb-select / mb-dropdown** (most occurrences) and **mb-search**.
   beta.71 has no `sl-menu-item type="checkbox"` option mode (newer Shoelace does).
   An attribute-override from medblocks was prototyped and **reverted** — Shoelace
   re-renders revert it, and partial overrides created new violations
   (`aria-required-parent`).
2. **Unnamed `sl-select` combobox input** (WCAG 4.1.2, 1.3.1) — ~12 controls
   across mb-quantity, mb-select, mb-text-select, mb-proportion, mb-percent.
   medblocks passes `label` to `sl-select`, but the beta's internal filter input
   does not expose it.
3. **`sl-checkbox` inner-input naming** (WCAG 4.1.2) — mb-checkbox,
   mb-checkbox-any. Likely the same beta limitation; confirm with a screen reader.
4. **Unit-label contrast** (WCAG 1.4.3) — mb-quantity, mb-proportion unit labels
   fail AA, via Shoelace `select__label` theming.

## Recommended decision (maintainers)
Because Shoelace is EOL, the durable options are all substantial:
- Upgrade to the final Shoelace **2.20.1** (years of a11y fixes; lands on a dead
  dependency), or
- Migrate to the **Web Awesome** successor (https://webawesome.com — check its
  licensing), or
- Reduce reliance on Shoelace for the a11y-critical list/select/checkbox controls.

These are not small patches and should be their own effort with full re-test.

Full detail: `documentation/accessibility/ISSUE-2-invalid-aria-menuitem.md` and
`ISSUE-3-contrast-unit-labels.md`.
Fingerprints (pattern): `A11Y-PAT-A4C1F4E77A3A` (menuitem),
`A11Y-PAT-96DE518F7A20` (contrast).

# [a11y] Several form controls have no accessible name

**Labels:** accessibility, bug

## Summary
Multiple `mb-*` form controls render without an accessible name, so screen
readers announce them as blank. This affects core data-entry controls. A fix for
the three standalone cases is available (see below); the rest trace to the
archived Shoelace beta.

## WCAG
4.1.2 Name, Role, Value (A); 1.3.1 Info and Relationships (A); 3.3.2 Labels or
Instructions (A)

## Affected (verified on the Kitchen Sink demo, axe-core + accessible-name computation)
Standalone (fixable in this repo, fix provided):
- **mb-multimedia** — the file input used `label=` (not a valid attribute on
  `<input>`); no accessible name, no visible label.
- **mb-duration** — per-unit inputs (Years/Months/Weeks/Days) set only
  `help-text` (→ `aria-describedby`), so each has a description but no name.
- **mb-buttons / mb-buttons-multiple** — a scaled-to-invisible proxy `<input>`
  (value-carrier for `reportValidity`) was keyboard-focusable and unnamed, a
  "edit, blank" stop next to the real choice buttons.

Shoelace-rooted (see separate issue / Shoelace note): mb-quantity, mb-select,
mb-text-select, mb-proportion, mb-percent (the `sl-select` internal combobox
input), and mb-checkbox / mb-checkbox-any.

## Steps to reproduce
1. `npm start`, open `http://localhost:8000/demo/demo.html` with a screen reader.
2. Tab to the Duration inputs, the file upload, or a buttons group.
3. The control is announced with no name.

## Expected
Each control exposes an accessible name equal to its visible label.

## Fix (provided, HTML-first)
Branch: https://github.com/mgifford/medblocks-ui/tree/a11y/accessible-names
- mb-multimedia: wrap the input in a native `<label>` (implicit association; no
  ARIA, no `id`/`for`).
- mb-duration: add `label` to each per-unit `sl-input`.
- mb-buttons/-multiple: add `tabindex="-1"` + `aria-hidden="true"` to the proxy
  input so it stays for validation but leaves the tab order and a11y tree.

Verified: axe `label` 20→4, `label-title-only` 16→2; no `aria-hidden-focus`
anti-pattern; keyboard focus confirmed to skip the proxy.

Full detail: `documentation/accessibility/ISSUE-1-missing-accessible-name.md`.
Fingerprint (pattern): `A11Y-PAT-7AA05807FF65`.

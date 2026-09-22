# [a11y] Dynamic form changes are not announced to screen readers

**Labels:** accessibility, enhancement

## Summary
The forms add and remove repeatable groups at runtime, but the library declares
no ARIA live region and does no focus management, so screen-reader users get no
feedback when the form structure changes. In a clinical form (e.g. adding
medication orders), silent changes are a safety concern.

## WCAG
4.1.3 Status Messages (AA); related 2.4.3 Focus Order (A)

## Evidence (verified on the demo)
Clicking "+ Order" on the auto-form:
```
inputs before: 66  →  after: 73   (7 new inputs added)
live regions on page: 0           (nothing announced)
focus after add: still on the "+" button (not moved to the new group)
```
`grep` for `aria-live` / `role="alert"` / `role="status"` / `role="log"` across
`src/` returns nothing. Add path: `src/medblocks/form/autoform-utils.ts`
(`insertBefore`); remove path (`removeChild`); repeatable components have no
focus/announcement logic.

## Steps to reproduce
1. Open the demo with a screen reader.
2. Activate "+ Order": new fields appear silently, focus does not move.
3. Activate a group's Delete: fields vanish silently, focus is lost.

## Expected
On add/remove, the change is announced (a polite live region, e.g. "Order 2
added") and/or focus is moved to a sensible target (first field/heading of the
new group; a stable element on remove).

## Suggested fix (framework-level, in mb-form)
1. A single visually-hidden `<div aria-live="polite" aria-atomic="true">` owned
   by `mb-form`, persistent in the DOM, whose text is updated on change (not
   `display:none`; create the region before the change).
2. Move focus to the new group's first field/heading on add.
3. Move focus to a stable target before removing on delete.
Use `polite`, not `assertive`, for routine structural changes.

Full detail: `documentation/accessibility/ISSUE-5-no-live-region-dynamic-form.md`.
Fingerprint (pattern): `A11Y-PAT-545EB8CED223`.

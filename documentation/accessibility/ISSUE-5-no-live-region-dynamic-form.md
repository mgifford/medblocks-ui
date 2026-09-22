## Accessibility Issue: Dynamic form changes are not announced to screen readers (no live region, no focus management)

**Bug ID:** `A11Y-OCC-F27B48D589A4` (occurrence) / `A11Y-PAT-545EB8CED223` (pattern)
**Fingerprint (pattern, a11y/pattern/v1):** `545eb8ced223b64892678fb0177a57696af6c089de46223f3d04f1b37bc9d952`
**Fingerprint (occurrence, a11y/occurrence/v1):** `f27b48d589a4d34f8f7d3206602ed00372c4f9207971a063b89a280cb7b31c1d`
**URL:** http://localhost:8000/demo/demo.html (medblocks-ui `master` @ 0.0.217)
**XPath (representative):** the repeatable container in `mb-form` / `mb-auto-form` (dynamic)
**WCAG SC:** 4.1.3 Status Messages (Level AA); related: 2.4.3 Focus Order (A), and general operable-form expectations for added/removed content
**Rule:** manual (semantic) + source review. Automated scanners cannot detect a MISSING live region (there is nothing to flag).
**Severity:** High
**Frequency:** Framework-wide. Affects every repeatable group in every form built with mb-form / mb-auto-form (add and remove), and any content shown/hidden dynamically. The demo's medication-order form is one instance.
**Screen type:** desktop | **Colour mode:** light

### Description

The forms are dynamic: repeatable groups are added and removed from the DOM at
runtime (for example the "+ Order" and "+ Medication order" buttons), and search
results and validation states change. **The library declares no ARIA live region
anywhere** (`aria-live`, `role="alert"`, `role="status"`, `role="log"` all absent
from source), and the repeatable add/remove code performs **no focus management**.

Consequences for a screen-reader user:
- Adding a group inserts new fields silently (`insertBefore`), with no
  announcement and no focus move. The user has no feedback that anything happened
  and must blindly hunt for the new fields.
- Removing a group deletes fields silently (`removeChild`) and can drop focus to
  `<body>`, disorienting the user.

In a clinical data-entry context (e.g. adding medication orders), silent DOM
changes are a safety concern: the user cannot be confident about what was added,
removed, or where they are.

### Evidence (verified on the live demo)

```
// Before/after clicking "+ Order" on the auto-form:
inputs_before: 66
inputs_after:  73        // 7 new inputs added
new_inputs_added: 7
live_regions_after_add: 0   // nothing announces the change
focus_after_add: "button"   // focus stayed on the + button, not moved to new group
```
Source: no live-region markup exists (`grep` for aria-live/role=alert/status/log
returns nothing). Add path: `src/medblocks/form/autoform-utils.ts:385`
(`insertBefore`). Remove path: `:305` (`removeChild`). Repeatable components
(`src/medblocks/repeat/*`) contain no focus or announcement logic.

### Steps to Reproduce

1. Open http://localhost:8000/demo/demo.html with a screen reader (NVDA/VoiceOver).
2. Navigate to the medication-order form and activate "+ Order".
3. Observe: a new group of fields appears, but nothing is announced and focus
   does not move.
4. Activate a group's Delete: fields are removed with no announcement; focus is
   lost.

### Expected Behaviour

When content is added or removed, the change is communicated to assistive
technology:
- A polite live region announces the change (e.g. "Order 2 added",
  "Order 2 removed"), AND/OR
- Focus is moved to a sensible target (e.g. the first field or heading of the
  newly added group on add; a stable nearby element on remove), so the user is
  taken to the change rather than left behind.

### Actual Behaviour

No announcement (no live region exists) and no focus management. The DOM changes
silently.

### Testing Environment

| Item | Value |
|---|---|
| Browser | Chromium (built-in browser pane), 2026-09-20 |
| OS | macOS (Darwin 27.0.0) |
| Screen reader | Pending manual NVDA/VoiceOver pass (finding from source + live DOM behaviour) |
| Testing tool | manual + DOM inspection (not scanner-detectable) |

### Impact

Screen-reader users (and, for the focus half, keyboard users) receive no feedback
when the form's structure changes. They cannot tell that fields were added or
removed, or navigate to them efficiently. Highest impact in the repeatable
clinical sections the library is designed for.

### Suggested Fix (framework-level)

This is best fixed once in the form/repeatable infrastructure rather than per
component:

1. **Add a visually-hidden polite live region** owned by `mb-form` (a single
   `<div aria-live="polite" aria-atomic="true">` in the form's shadow root, kept
   off-screen with the standard visually-hidden pattern, NOT `display:none`).
   On add/remove, write a short message into it ("Order 2 added." /
   "Order 2 removed."). Use `polite`, not `assertive`, for these
   non-urgent structural changes.
2. **Move focus on add**: after inserting a new repeatable group, move focus to
   its first focusable field or its heading (give the heading `tabindex="-1"`).
3. **Handle focus on remove**: before removing a group, move focus to a stable
   target (the preceding group's heading, or the "+ Add" button), so focus is
   never lost to `<body>`.

Notes for a robust implementation:
- The live region must exist in the DOM BEFORE the change and must be a
  persistent node whose text content is updated (creating the region and its text
  in the same tick often fails to announce).
- Keep messages short and specific; include the group name/index.
- Do not use `role="alert"`/`assertive` for routine add/remove; reserve assertive
  for genuine errors.

### Verification required (name/role/state + interaction class)

Per the a11y-test evidence contract, this needs interaction evidence, not a
screenshot:
- A screen-reader / emulated-SR check that the live region announces on add and
  remove (`live_announcements` captured), AND
- A real-keyboard trace showing focus moves to the new group on add and to a
  stable target on remove.

### Related
- Focus management overlaps 2.4.3 Focus Order.
- Independent of the labelling fixes (ISSUE-1) and the invalid-ARIA search
  (ISSUE-2); this is the dynamic-update layer.

## Accessibility Issue: Components emit duplicate `id` values across instances

**Bug ID:** `A11Y-OCC-6541319BA496` (occurrence) / `A11Y-PAT-D8725C3023EB` (pattern)
**Fingerprint (pattern, a11y/pattern/v1):** `d8725c3023eb36004066568875373bd14dc62ce7cc73fdece90ddce57bb30343`
**Fingerprint (occurrence, a11y/occurrence/v1):** `6541319ba4964e8006513808afe7f302c694808da3aaafcb3944c0c8301642e3`
**URL:** http://localhost:8000/demo/demo.html (medblocks-ui `master` @ 0.0.217)
**XPath (representative):** `//input[@id="input"]` (matches many)
**WCAG SC:** 4.1.1 Parsing (removed in WCAG 2.2, but duplicate ids break `for`/`aria-labelledby`/`aria-describedby` association, which is 1.3.1 / 4.1.2 territory)
**Rule:** axe-core 4.6.3 — `duplicate-id` (minor)
**Severity:** Medium (elevated from Low because it undermines the labelling fix in ISSUE-1)
**Frequency:** 37 axe nodes; 9 genuinely repeated ids in the light DOM: `input` x3, `input_multiple` x3, `search` x3, `quantity` x3, `select` x4, `proportion` x2, `duration` x5, `date` x3, `checkbox` x2.
**Screen type:** desktop | **Colour mode:** light

### HTML Snippet

```html
<!-- Same id repeated on multiple instances; source defaults a fixed id -->
<!-- e.g. src/medblocks/quantity/quantity.ts:89  @property() id = 'quantity'; -->
<mb-quantity id="quantity" ...></mb-quantity>
<mb-quantity id="quantity" ...></mb-quantity>
```

### Description

Several components default their host or inner `id` to a fixed string (for
example mb-quantity defaults `id = 'quantity'`, and inner controls use
`id="input"`). When more than one instance is placed on a page, ids collide.

Some of axe's 37 flagged nodes are inside separate shadow roots (ids are scoped
per shadow root, so those are arguable false positives). But 9 collisions are in
the light DOM and are real. Duplicate ids matter here beyond the obsolete 4.1.1:
if the ISSUE-1 fix uses `aria-labelledby` / `<label for>`, those associations
depend on unique ids, so this should be fixed together with or before ISSUE-1.

### Steps to Reproduce

1. Open http://localhost:8000/demo/demo.html (which places multiple instances).
2. Confirm real light-DOM collisions:
   ```js
   const ids={}; document.querySelectorAll('[id]').forEach(e=>ids[e.id]=(ids[e.id]||0)+1);
   Object.entries(ids).filter(([k,n])=>n>1)
   // => input:3, duration:5, select:4, quantity:3, date:3, ...
   ```

### Expected Behaviour

Each instance produces unique ids (or does not rely on fixed ids), so label and
description associations resolve to the intended single element.

### Actual Behaviour

Multiple instances share the same id; associations are ambiguous.

### Testing Environment

| Item | Value |
|---|---|
| Browser | Chromium (built-in browser pane), 2026-09-20 |
| OS | macOS (Darwin 27.0.0) |
| Screen reader | N/A (structural) |
| Testing tool | axe-core 4.6.3 |

### Impact

Ambiguous id references can cause a label/description to associate with the wrong
element, or none, undermining the accessible-name work in ISSUE-1. Primarily
affects screen-reader users.

### Suggested Fix

Generate unique ids per instance instead of fixed defaults. Options:
- Use a per-instance unique suffix (e.g. a counter or a short random/UUID
  fragment) when building inner control ids.
- Or scope all id-based associations within the component's own shadow root and
  avoid light-DOM fixed ids.

Verification: re-scan (`duplicate-id` light-DOM collisions -> 0) and confirm
label associations still resolve.

### Evidence

- `notes/a11y-evidence/axe-kitchen-sink-2026-09-20.json` (rule `duplicate-id`)
- Source: `src/medblocks/quantity/quantity.ts:89` (fixed `id='quantity'`) and the
  inner `id="input"` usage across text/quantity/duration components.

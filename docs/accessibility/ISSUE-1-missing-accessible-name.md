## Accessibility Issue: Form controls have no accessible name (label rendered as unassociated `<p>`)

**Bug ID:** `A11Y-OCC-E5924ADD7EB7` (occurrence) / `A11Y-PAT-7AA05807FF65` (pattern)
**Fingerprint (pattern, a11y/pattern/v1):** `7aa05807ff65b59e59dde251644f851fcfd727f7fc01509e44af0f326642747c`
**Fingerprint (occurrence, a11y/occurrence/v1):** `e5924add7eb7b64f16c85161e56d076b428ca703cf7f0c12e3b01bc8444376c0`
**URL:** http://localhost:8000/demo/demo.html (Kitchen Sink demo, medblocks-ui `master` @ 0.0.217)
**XPath (representative):** `//mb-duration[@path="duration/ym"]` -> shadow -> `#duration-Years` -> `#input`
**Full DOM path (representative):** `/html/body/div/mb-form/mb-duration/#shadow-root/div/sl-input/#shadow-root/div/div/div/input`
**WCAG SC:** 4.1.2 Name, Role, Value (Level A); 1.3.1 Info and Relationships (Level A); 3.3.2 Labels or Instructions (Level A)
**Rule:** axe-core 4.6.3 — `label` (critical), `label-title-only` (serious); confirmed by manual accessible-name computation
**Severity:** Critical
**Frequency:** 19 of 66 native form controls on this page, spanning 10 components: mb-quantity (5), mb-select (4), mb-multimedia (3), mb-text-select, mb-buttons, mb-buttons-multiple, mb-proportion, mb-percent, mb-checkbox, mb-checkbox-any. Single shared root cause.
**Screen type:** desktop | **Colour mode:** light

### HTML Snippet

```html
<!-- The visible label is a bare <p>, not associated with any control: -->
<!-- src/medblocks/EhrElement.ts:86 -->
<p style="font-weight: 600;">Dose</p>
<!-- ...elsewhere in shadow DOM, the actual control has no accessible name: -->
<input part="input" id="input" class="input__control" type="number" aria-describedby="help-text">
```

### Description

The shared base-class helper `EhrElement._label()` renders each component's
`label` property as a standalone paragraph:

```ts
// src/medblocks/EhrElement.ts:86
_label() {
  if (this.label) return html`<p style="font-weight: 600;">${this.label}</p>`;
  return undefined;
}
```

A `<p>` is a visual label only. It is not linked to the input via `<label for>`,
`aria-labelledby`, or `aria-label`, so the native control that assistive
technology focuses has no accessible name. A screen-reader user tabbing through
an affected form hears "edit text, blank" (or the control type with no name),
and cannot tell which field they are in. Voice-control users cannot address the
field by name.

Because the defect is in one shared helper, a single fix resolves it across all
affected components.

### Steps to Reproduce

1. Serve the library demo: `npm start` in the medblocks-ui repo (opens :8000).
2. Open http://localhost:8000/demo/demo.html
3. Open a screen reader (NVDA + Firefox, or VoiceOver + Safari).
4. Tab to the "Duration" component's Years/Months inputs, or the "Dose"
   quantity, or any mb-select/mb-buttons field.
5. Listen: the control is announced with no name (e.g. "edit, blank" /
   "spin button, blank").

Automated confirmation:
```js
// injected axe-core 4.6.3 on the page
axe.run(document, { runOnly: ['label','label-title-only'] })
// => label: 20 nodes (critical); label-title-only: 16 nodes (serious)
```

### Expected Behaviour

Every form control exposes an accessible name equal to its visible label. A
screen reader announces, for example, "Dose, edit" or "Years, spin button".

### Actual Behaviour

The control has no accessible name. The visible `<p>` label is not
programmatically associated with it.

### Testing Environment

| Item | Value |
|---|---|
| Browser | Chromium (built-in browser pane), 2026-09-20 |
| OS | macOS (Darwin 27.0.0) |
| Screen reader | Pending manual NVDA/VoiceOver pass (finding derived from accessible-name computation + axe) |
| Testing tool | axe-core 4.6.3 |

### Impact

Blind and low-vision screen-reader users cannot identify affected fields.
Voice-control users cannot target them by name. This affects most data-entry
controls the library produces, so it undermines the library's core purpose.

### Suggested Fix

Associate the label with the control. Minimal, low-risk options:

Option A — give the `<p>` an id and point the control at it with
`aria-labelledby` (works across the shadow boundary only if the control and the
`<p>` share a shadow root; where the control is inside a nested Shoelace element,
forward the name instead):

```ts
_label(id = 'mb-label') {
  if (this.label) return html`<p id=${id} style="font-weight: 600;">${this.label}</p>`;
  return undefined;
}
```

Option B (more robust for the Shoelace-wrapped controls like mb-quantity,
mb-input, mb-duration): pass the label into the inner `sl-input`/`sl-select`
`label` slot consistently (some components already do this; the ones in the
frequency list do not), OR set `aria-label=${this.label}` on the inner control.

Verification note: fixing this needs a re-scan (axe `label` count -> 0) AND a
real screen-reader check that each control announces its name, since the
required evidence for a name/role/state fix is computed AT output, not a
screenshot.

### Scope correction after source investigation (2026-09-20)

The "10 components, one root cause" framing was too simple. Verified breakdown of
the 19 unnamed controls (`notes/a11y-evidence/unnamed-controls-breakdown-2026-09-20.json`):
- **Clear medblocks source bugs (fixed in prototype):** mb-multimedia file input
  (invalid `label` attr) and mb-duration per-unit inputs (no `label`, only
  `help-text`). Prototype dropped axe `label` 20->4 and `label-title-only` 16->2.
- **Primarily Shoelace-beta (not a simple medblocks fix):** 12 controls are
  `sl-select`'s internal combobox input (mb-quantity, mb-select, mb-text-select,
  mb-proportion, mb-percent); 2 are `sl-checkbox` inner inputs.
- **mb-buttons/mb-buttons-multiple hidden proxy input:** a medblocks item, but
  the fix is likely to remove it from the a11y tree, not label it.

So this issue is best split: (1) a tight PR for mb-multimedia + mb-duration
(done in prototype), and (2) a separate discussion of the Shoelace `sl-select`
naming gap. See `PRIORITIZATION.md`.

### Evidence

- `notes/a11y-evidence/accessible-name-analysis-2026-09-20.json`
- `notes/a11y-evidence/unnamed-controls-breakdown-2026-09-20.json`
- `notes/a11y-evidence/axe-kitchen-sink-2026-09-20.json`
- `notes/a11y-evidence/prototype-fix-2026-09-20.diff`
- Source: `src/medblocks/EhrElement.ts:86`, `multimedia/multimedia.ts:113`,
  `duration/duration.ts:174`

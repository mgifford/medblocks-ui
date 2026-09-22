## Accessibility Issue: Unit labels fail colour contrast (mb-quantity, mb-proportion)

**Bug ID:** `A11Y-OCC-CD66A12BAAF2` (occurrence) / `A11Y-PAT-96DE518F7A20` (pattern)
**Fingerprint (pattern, a11y/pattern/v1):** `96de518f7a20c469437314c8b9a629eedd380b499bfaecc9ea17f762e8cdfdcb`
**Fingerprint (occurrence, a11y/occurrence/v1):** `cd66a12baaf21cdbce585621186f0ff9b0365f10c77a54ad20f64cc5acbb9352`
**URL:** http://localhost:8000/demo/demo.html (medblocks-ui `master` @ 0.0.217)
**XPath (representative):** `//mb-quantity[@path="quantity/2"]//div[@part="display-label"]`
**Full DOM path (representative):** `/html/body/div/mb-form/mb-quantity/#shadow-root/sl-select/#shadow-root/div/div/sl-dropdown/div/div`
**WCAG SC:** 1.4.3 Contrast (Minimum) (Level AA)
**Rule:** axe-core 4.6.3 — `color-contrast` (serious)
**Severity:** Medium
**Frequency:** 5 nodes: mb-quantity unit label ("mmHg"), mb-proportion unit label ("/ 1"), plus 3 slate section headings (the latter are `mb-auto-form` styles, borderline component/scaffold).
**Screen type:** desktop | **Colour mode:** light

### HTML Snippet

```html
<!-- Unit display label, low contrast against its background -->
<div part="display-label" class="select__label">mmHg</div>
<div part="display-label" class="select__label">/ 1</div>
```

### Description

The unit display label inside mb-quantity and mb-proportion (rendered via
Shoelace `sl-select` display-label) does not meet the WCAG AA 4.5:1 text
contrast minimum. Because this is text inside a UI component, the text threshold
(4.5:1) applies, not the 3:1 non-text component threshold. Low-vision users and
users in bright environments may not be able to read the unit, which in a
clinical form is safety-relevant (mmHg vs other pressure units).

Note: the exact measured ratios were not extracted in this pass; axe flags them
as failing. A precise ratio per node should be recorded before filing upstream.

### Steps to Reproduce

1. Open http://localhost:8000/demo/demo.html
2. Find "Blood Pressure (mb-quantity with only one unit)" and the mb-proportion
   field. Observe the greyed unit label.
3. Confirm:
   ```js
   axe.run(document, { runOnly: ['color-contrast'] })
   // => 5 nodes incl. .select__label[part=display-label]
   ```

### Expected Behaviour

Unit labels meet at least 4.5:1 contrast against their background.

### Actual Behaviour

The unit label text is below 4.5:1 (exact ratio to be measured).

### Testing Environment

| Item | Value |
|---|---|
| Browser | Chromium (built-in browser pane), 2026-09-20 |
| OS | macOS (Darwin 27.0.0) |
| Screen reader | N/A (visual criterion) |
| Testing tool | axe-core 4.6.3 |

### Impact

Low-vision users, and any user in poor lighting, may misread or fail to read the
measurement unit on clinical data-entry fields.

### Suggested Fix

Increase the unit label colour to meet 4.5:1. This lives in the component/Shoelace
theming (the `select__label` / `display-label` part). Options: darken the label
text via the component's CSS custom properties / `::part(display-label)` styling,
or set an explicit accessible token. Confirm in both light and (if supported)
dark modes.

Verification: re-scan (`color-contrast` -> 0 for these nodes) and a screenshot
comparison (visual-class evidence is appropriate here).

### Evidence

- `notes/a11y-evidence/axe-kitchen-sink-2026-09-20.json` (rule `color-contrast`)
- Source: `src/medblocks/quantity/` and `src/medblocks/proportion/`, plus the
  Shoelace `select` part theming.

### Follow-up before filing
- [ ] Extract the exact contrast ratio per node (axe `color-contrast` data
      includes fg/bg/ratio) so the report states measured values.

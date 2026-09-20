## Accessibility Issue: Invalid ARIA — `aria-checked` on `role="menuitem"` in search/dropdown results

**Bug ID:** `A11Y-OCC-BBC28F20778C` (occurrence) / `A11Y-PAT-A4C1F4E77A3A` (pattern)
**Fingerprint (pattern, a11y/pattern/v1):** `a4c1f4e77a3a7f417591f42f17fbc6f3ef092b415775e3bfba6adcf43e7e05e9`
**Fingerprint (occurrence, a11y/occurrence/v1):** `bbc28f20778c516db020e6c8191b4d99746200e561663b10e476797d4d465c1b`
**URL:** http://localhost:8000/demo/demo.html (medblocks-ui `master` @ 0.0.217)
**XPath (representative):** `//mb-search//sl-menu-item[@role="menuitem"][@aria-checked]`
**Full DOM path (representative):** `/html/body/div/mb-form/mb-search/sl-menu-item[1]`
**WCAG SC:** 4.1.2 Name, Role, Value (Level A)
**Rule:** axe-core 4.6.3 — `aria-allowed-attr` (critical). ACT: `5c01ea` (ARIA state or property is permitted).
**Severity:** High
**Frequency:** 6 nodes across mb-search, mb-search-multiple, mb-dropdown result menus. Shared root cause in the coded-text result rendering.
**Screen type:** desktop | **Colour mode:** light

### HTML Snippet

```html
<!-- role="menuitem" does not permit aria-checked -->
<sl-menu-item value="Something" slot="results"
              aria-disabled="false" aria-checked="false"
              role="menuitem" tabindex="0">Okayyy</sl-menu-item>
```

Also on the dropdown panel:
```html
<div part="panel" class="dropdown__panel" aria-labelledby="dropdown" aria-hidden="false">
```
`aria-labelledby="dropdown"` references id `dropdown`; confirm that id exists in
the same shadow root (if absent, the panel has a dangling reference — a second,
related defect).

### Description

The coded-text search components render result options as `sl-menu-item` with
`role="menuitem"` but also set `aria-checked`. Per ARIA, `aria-checked` is not an
allowed attribute on `role="menuitem"`; it is allowed on `menuitemcheckbox` and
`menuitemradio`. Assistive technology may ignore the state or expose a confusing
"menu item" that also claims a checked state, so users cannot reliably tell which
results are selected in a multi-select search.

### Steps to Reproduce

1. Open http://localhost:8000/demo/demo.html
2. Scroll to the "Route" (mb-search) or "Chief complaints" (mb-search-multiple)
   search fields and type to open the results menu.
3. Inspect a result item, or run:
   ```js
   axe.run(document, { runOnly: ['aria-allowed-attr'] })
   // => 6 nodes: sl-menu-item[role=menuitem][aria-checked]
   ```

### Expected Behaviour

Selectable results expose a role that permits a checked state
(`menuitemcheckbox` for multi-select, `menuitemradio` for single-select), so
assistive technology announces "menu item checkbox, checked/not checked".

### Actual Behaviour

Results use `role="menuitem"` with a disallowed `aria-checked`, an invalid
ARIA combination.

### Testing Environment

| Item | Value |
|---|---|
| Browser | Chromium (built-in browser pane), 2026-09-20 |
| OS | macOS (Darwin 27.0.0) |
| Screen reader | Pending manual NVDA/VoiceOver pass |
| Testing tool | axe-core 4.6.3 |

### Impact

Screen-reader users of the coded-text search (a core openEHR data-entry control)
get an invalid or missing selection state, so they cannot reliably tell which
options are chosen, especially in multi-select.

### Scope + root-cause correction (2026-09-20, after investigation)

Two corrections from source reading and live testing:

1. **Origin is Shoelace, not medblocks source.** `role="menuitem"` and
   `aria-checked` are set by Shoelace's `sl-menu-item` itself (beta.71), not by
   medblocks code. medblocks just renders `<sl-menu-item>`. beta.71 has no
   `type="checkbox"`/option mode (newer Shoelace does), so medblocks cannot fix
   it at the source. **[verified]** grep of codedtext/ (no role/aria-checked);
   live DOM (`sl-menu-item` self-assigns them; `type` prop absent).
2. **Most occurrences are NOT search.** Of the 25 `sl-menu-item`s on the demo,
   24 belong to **mb-select / mb-dropdown** (the select family, which reuse
   `sl-menu`), and only 1 was in an `mb-search`. So the `aria-allowed-attr`
   nodes are mostly the select components, not search. **[verified]** live
   owner-attribution.

**Override attempt (reverted).** Overriding to `role="option"` + `aria-selected`
via a lifecycle hook + shadow-root MutationObserver was prototyped and
**reverted**: Shoelace re-renders results asynchronously on each keystroke and
reverts the roles, and a partial override created new violations
(`aria-required-parent` from an orphaned option). Attribute-overriding
Shoelace's internally-managed roles is not robust. **[verified]** live retest.

### Suggested Fix (revised)

This is a Shoelace-beta root cause, in the same class as the `sl-select` naming
gap. Robust options, in order of preference:
- **Upgrade Shoelace** to a version with `sl-menu-item type="checkbox"` and/or a
  proper combobox/listbox option pattern, then use it. (Larger change; affects
  the whole library's Shoelace dependency.)
- **Rework the search/select list rendering** to a real combobox+listbox
  (`role="combobox"` input with `aria-expanded`/`aria-controls`/
  `aria-activedescendant`, `role="listbox"` container, `role="option"` items)
  built on primitives medblocks controls, not `sl-menu`. Substantial.
- Do NOT ship an attribute-override patch: verified unreliable.

Also verify the `aria-labelledby="dropdown"` target id exists in the panel's
shadow root; if not, set a real label or remove the reference.

Verification: re-scan (`aria-allowed-attr` -> 0) AND a screen-reader check that
result selection state is announced correctly (name/role/state evidence, not a
screenshot).

### Evidence

- `notes/a11y-evidence/axe-kitchen-sink-2026-09-20.json` (rule `aria-allowed-attr`)
- Source to inspect: `src/medblocks/codedtext/` (search.ts, dropdown.ts, option.ts, abstractSearch.ts)

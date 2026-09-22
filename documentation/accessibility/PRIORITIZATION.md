# Upstream accessibility issues: prioritization and PR guidance

Which component-level accessibility defects to report first, and the concrete
source fix that could accompany each as a pull request. Scope: fixes inside
medblocks-ui source (`master` @ 0.0.217). Evidence in `notes/a11y-evidence/`;
per-issue reports in this directory. All claims **[verified]** against source
unless marked **[inferred]**.

## Ranking

| Rank | Issue | Severity | Reach | PR difficulty | Why this order |
|---|---|---|---|---|---|
| 1 | ISSUE-1 accessible names | Critical | ~10 components, 19 controls | Low-Medium | Biggest user impact; the library's core job is data entry, and unnamed fields defeat screen-reader use. Fix is small and mostly per-component. |
| 2 | ISSUE-2 invalid ARIA on menuitems | High | 3 search/dropdown components | Low | Invalid ARIA on the coded-text search, a core openEHR control. Role swap is contained. |
| 3 | ISSUE-4 duplicate ids | Medium | 9 components | Low-Medium | Should land with/before ISSUE-1 because label association needs unique ids. |
| 4 | ISSUE-3 contrast on unit labels | Medium | 2 components | Low | Real but narrower; needs exact ratios measured first. |

Recommendation: open **ISSUE-1 first, with a PR**, because it has the highest
impact and the fix is well understood. ISSUE-4 pairs naturally with it. ISSUE-2
is a strong, small, independent second PR.

## ISSUE-1 is not one bug; it is one root cause with several fix sites

**[verified]** The shared helper renders an unassociated `<p>`:
```ts
// src/medblocks/EhrElement.ts:86
_label() {
  if (this.label) return html`<p style="font-weight: 600;">${this.label}</p>`;
  return undefined;
}
```
16 components call `_label()`. But the actual failing controls fall into three
distinct fix patterns, so a single `_label()` change does not fix all of them.
The investigation found:

### Pattern A: control is a raw native input with an invalid `label` attribute
- **mb-multimedia** (`src/medblocks/multimedia/multimedia.ts:113-120`) **[verified]**:
  renders `<input type="file" ... label=${this.label}>`. `label` is **not a valid
  attribute on `<input>`**, so it does nothing; the file input is nameless, and
  there is no visible label either (it is wrapped in a bare `<p>`).
  **Fix (small, concrete):** replace `label=` with `aria-label=${this.label}` on
  the file input, and/or render an associated `<label for="file">`. One-line
  class change.

### Pattern B: control is a Shoelace input missing its `label`, using only `help-text`
- **mb-duration** (`src/medblocks/duration/duration.ts:165-178`) **[verified]**:
  each per-unit `sl-input` (Years/Months/Weeks/Days) sets `help-text` (which maps
  to `aria-describedby`) but **no `label`**, so each spinbutton has a description
  but no name. The component's outer `<label>` (line 193) is a group label, not
  tied to each input.
  **Fix:** add `label=${this.formatDuration(a)}` (or `aria-label`) to each
  `sl-input` in the `.map()`. Small, contained.

### Pattern C: control relies only on the `_label()` `<p>`
- Components that render `${this._label()}` next to a control that is not a
  Shoelace element carrying its own `label` (candidates: mb-select, mb-buttons,
  mb-text-select, mb-checkbox and the multiple variants). **[inferred]** for the
  exact per-component wiring; needs a per-component read before the PR.
  **Fix:** make `_label()` associate the `<p>` with the control, e.g. give the
  `<p>` a stable id and set `aria-labelledby` on the control (same shadow root),
  or forward `aria-label`. This is the shared-helper part of the fix.

**Consequence for the PR:** ISSUE-1 is best delivered as a small PR touching
`EhrElement._label()` (Pattern C) plus the two concrete component fixes
(multimedia, duration). That is a coherent, reviewable change with a clear
before/after. Do NOT claim one line fixes all 10 components; it does not.

**Dependency:** Pattern C's `aria-labelledby` approach needs unique ids
(ISSUE-4). If the PR uses `aria-label` instead, it avoids the id dependency.
**[inferred]** `aria-label` is the lower-risk route for a first PR.

## ISSUE-2 fix sketch (second PR candidate)

**[verified]** search results render `role="menuitem"` with a disallowed
`aria-checked`. Fix in `src/medblocks/codedtext/` (search.ts / search-multiple.ts
/ option.ts / dropdown.ts): set `role="menuitemcheckbox"` (multi) or
`menuitemradio` (single) on selectable results so `aria-checked` is valid. Also
verify the dropdown panel's `aria-labelledby="dropdown"` target exists.

## Verification required before any PR is "done"

Per the a11y-test evidence contract, evidence type must match the defect class:
- ISSUE-1, ISSUE-2 (name/role/state): re-scan axe (rule count -> 0) AND a real
  screen-reader check that each control announces its name/state. A screenshot is
  NOT sufficient.
- ISSUE-3 (contrast, visual): re-scan + screenshot comparison; extract exact
  ratios first.
- ISSUE-4 (structural): re-scan (light-DOM duplicate ids -> 0) + confirm label
  associations still resolve.

## Prototype result (verified before/after, 2026-09-20)

A prototype fix was built on branch `a11y/accessible-names-prototype` in the
clone (2 files, 6 lines; diff at `notes/a11y-evidence/prototype-fix-2026-09-20.diff`):
- **mb-multimedia** (`multimedia.ts`): `label=` (invalid on `<input>`) changed to
  `aria-label=`, plus an associated `<label for="file">`.
- **mb-duration** (`duration.ts`): added `label=${this.formatDuration(a)}` to each
  per-unit `sl-input`.

Rebuilt (`npm run build`, clean) and re-scanned the demo. **[verified]** live axe:

| Metric | Before | After |
|---|---|---|
| axe `label` nodes | 20 | **4** |
| axe `label-title-only` nodes | 16 | **2** |
| Unnamed native controls | 19 | **16** |
| mb-multimedia file inputs named | 0/3 | **3/3** (aria-label) |
| mb-duration number inputs named | 0/16 | **16/16** (label-for) |

So two small, clean source changes fixed ~30 of ~36 label-rule node failures.

**Honest scope correction (important):** the remaining 16 unnamed controls are
NOT simple medblocks source bugs. **[verified]** live analysis
(`notes/a11y-evidence/unnamed-controls-breakdown-2026-09-20.json`):
- 12 are **Shoelace `sl-select`'s internal combobox input** (mb-quantity x5,
  mb-select x4, mb-text-select, mb-proportion, mb-percent). medblocks passes
  `label` to `sl-select`, but the beta.71 internal input does not expose it.
  Primarily a Shoelace-beta issue; mitigable but not fixed by `_label()`.
- 2 are **sl-checkbox** inner inputs (mb-checkbox, mb-checkbox-any) — likely
  Shoelace-beta name wiring; verify with a real screen reader.
- 2 are the **mb-buttons/mb-buttons-multiple hidden proxy input** — a medblocks
  source item, but the right fix may be to remove it from the a11y tree
  (`aria-hidden` / not focusable) rather than label a scaled-to-zero input.

This corrects the earlier "one shared root cause across 10 components" framing:
the clean, high-confidence medblocks-ui PR is mb-multimedia + mb-duration. The
`sl-select` group is a larger, separate conversation (Shoelace upgrade or
mitigation) worth raising with the founder rather than filing as a simple bug.

## Suggested contribution sequence

1. Confirm Pattern C per-component wiring (short source read) so the PR scope is
   exact.
2. Open ISSUE-1 (report) + PR: `_label()` association + mb-multimedia + mb-duration.
   Use `aria-label` to avoid the id dependency for v1.
3. Open ISSUE-2 (report) + PR: menuitem role fix.
4. ISSUE-4 and ISSUE-3 as follow-ups.

Note: the repo has no CONTRIBUTING file or PR template, and last commit was
2025-04-29, so response latency is unknown. Raising ISSUE-1 with the founder
meeting is a good way to gauge appetite before investing in more PRs.

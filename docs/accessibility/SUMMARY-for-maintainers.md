# medblocks-ui accessibility findings

A consolidated accessibility review of medblocks-ui, prepared for the
maintainers. WCAG 2.2 AA is the reference standard. Findings are grouped by
whether they can be fixed inside medblocks-ui or stem from the archived Shoelace
dependency.

- Reviewed: `master` @ 0.0.217 (repo tip), the Kitchen Sink demo (`demo/demo.html`)
  and a minimal blood-pressure form. Synthetic data only.
- Method: axe-core 4.6.3 (automated), deep shadow-DOM accessible-name analysis,
  real keyboard testing, and source review. Automated tooling covers ~30-40% of
  WCAG; the rest is manual. Each finding is labelled verified (tool/source) or
  inferred.
- Fingerprints follow the a11y/pattern/v1 and a11y/occurrence/v1 profiles
  (https://mgifford.github.io/ACCESSIBILITY.md/examples/fingerprints/) for stable
  dedup/tracking.
- Verified fixes for the standalone items are on a branch:
  https://github.com/mgifford/medblocks-ui/tree/a11y/accessible-names

## Context that shapes the recommendations: Shoelace is end-of-life

medblocks-ui depends on `@shoelace-style/shoelace@2.0.0-beta.71`, a pre-1.0
beta. As of this review the Shoelace project is **archived** on GitHub (its
description now reads "Shoelace is now Web Awesome"); the final release was
2.20.1 (March 2025). Several of the findings below originate inside Shoelace
components, not medblocks source, and cannot be fixed cleanly by medblocks while
on the beta. The strategic options (all substantial, all maintainer decisions):
upgrade to the final Shoelace 2.20.1, migrate to the Web Awesome successor
(check its licensing), or reduce reliance on Shoelace for a11y-critical controls.

## Group A: fixable in medblocks-ui (fixes provided, verified)

These do not depend on Shoelace. Fixes are committed on the branch above and
verified (axe `label` 20->4, `label-title-only` 16->2; no new violations).

| ID | Finding | WCAG | Severity | Fix commit |
|---|---|---|---|---|
| ISSUE-1 (part) | mb-multimedia file input unnamed (`label=` is invalid on `<input>`) | 4.1.2, 1.3.1, 3.3.2 | Critical | 5f1023f (wrapped `<label>`) |
| ISSUE-1 (part) | mb-duration unit inputs unnamed (only `help-text`, no label) | 4.1.2, 1.3.1 | Critical | c818388 |
| (from ISSUE-1) | mb-buttons/-multiple nameless hidden proxy input, keyboard-focusable | 4.1.2, 2.4.3 | High | 181db3e (removed from a11y tree) |

Details: `ISSUE-1-missing-accessible-name.md`. Fixes are HTML-first (native
`<label>`, not ARIA, per the "use HTML before ARIA" principle) and were verified
with real keyboard testing and accessible-name computation.

## Group B: framework-level, fixable in medblocks-ui (no patch yet)

| ID | Finding | WCAG | Severity |
|---|---|---|---|
| ISSUE-5 | Dynamic add/remove of repeatable groups is not announced (no ARIA live region anywhere in the library) and no focus management | 4.1.3, 2.4.3 | High |

This is the most important framework gap for a clinical form: adding a
medication-order group inserts 7+ fields silently with no announcement and no
focus move (verified live). Fix is a single polite live region owned by
`mb-form` plus focus management in the repeatable add/remove paths. Details:
`ISSUE-5-no-live-region-dynamic-form.md`. Offered as a follow-up PR if the
maintainers want it.

## Group C: rooted in the archived Shoelace beta (report only)

These originate in Shoelace components. medblocks renders them correctly; the
invalid semantics come from beta.71. Attribute-overriding from medblocks was
prototyped for the search case and proved unreliable (Shoelace re-renders revert
it), so no patch is offered. The durable fix is the Shoelace decision above.

| ID | Finding | WCAG | Severity | Origin |
|---|---|---|---|---|
| ISSUE-2 | `aria-checked` on `role="menuitem"` (invalid) in result/option lists | 4.1.2 | High | `sl-menu-item` beta.71; affects mb-select/mb-dropdown (most) and mb-search |
| (naming) | `sl-select` internal combobox input unnamed (~12 controls: mb-quantity, mb-select, mb-text-select, mb-proportion, mb-percent) | 4.1.2, 1.3.1 | High | `sl-select` beta.71 |
| (naming) | `sl-checkbox` inner input naming (mb-checkbox, mb-checkbox-any) | 4.1.2 | Medium | `sl-checkbox` beta.71 |
| ISSUE-3 | Unit labels fail colour contrast (mb-quantity, mb-proportion) | 1.4.3 | Medium | Shoelace `select__label` theming (part) |

Details: `ISSUE-2-invalid-aria-menuitem.md`, `ISSUE-3-contrast-unit-labels.md`.

## Group D: usage/structural (lower priority)

| ID | Finding | WCAG | Severity |
|---|---|---|---|
| ISSUE-4 | Components emit duplicate `id`s across instances (fixed default ids) | 4.1.1 (obsolete) / affects 1.3.1, 4.1.2 associations | Medium |

Details: `ISSUE-4-duplicate-ids.md`. Works today only because shadow-root id
scoping; fragile. Worth fixing alongside any label work that uses id/for.

## Not yet covered (gaps in this review, for transparency)

- Real screen-reader passes (NVDA/VoiceOver); findings are from axe + computed
  accessible-name + real keyboard, not yet AT-confirmed.
- Error-state / required-field announcement testing.
- Target size (2.5.8) and 320px reflow (1.4.10).
- Exact contrast ratios for ISSUE-3 (axe flags; values not extracted).

## Recommended contribution sequence

1. Merge Group A (branch `a11y/accessible-names`) — small, verified, no Shoelace
   dependency.
2. Decide Group B (live regions) — offer a follow-up PR.
3. Treat Group C as input to the Shoelace strategic decision, not as small bugs.
4. Group D alongside any id-dependent labelling.

## Suggested infrastructure (see the two TODOs)

To catch a11y regressions before commit: add axe-core to the existing
`@web/test-runner` suite plus a pre-commit/CI check; and add virtual
assistive-technology tests with Guidepup (https://www.guidepup.dev/) for
screen-reader-level coverage beyond axe.

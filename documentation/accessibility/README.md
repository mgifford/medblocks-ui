# Accessibility findings for medblocks-ui

A WCAG 2.2 AA review of the `mb-*` web components. Start with the summary, then
the per-issue detail. Ready-to-file GitHub issue drafts are in
[`github-issues/`](github-issues/).

- **[SUMMARY-for-maintainers.md](SUMMARY-for-maintainers.md)** — start here.
  Groups findings by fixability and notes that Shoelace (beta.71) is archived/EOL.
- [ISSUE-1-missing-accessible-name.md](ISSUE-1-missing-accessible-name.md)
- [ISSUE-2-invalid-aria-menuitem.md](ISSUE-2-invalid-aria-menuitem.md)
- [ISSUE-3-contrast-unit-labels.md](ISSUE-3-contrast-unit-labels.md)
- [ISSUE-4-duplicate-ids.md](ISSUE-4-duplicate-ids.md)
- [ISSUE-5-no-live-region-dynamic-form.md](ISSUE-5-no-live-region-dynamic-form.md)
- [PRIORITIZATION.md](PRIORITIZATION.md)

Verified fixes for the standalone items:
https://github.com/mgifford/medblocks-ui/tree/a11y/accessible-names

Method: axe-core (automated), deep shadow-DOM accessible-name analysis, real
keyboard testing, source review. Automated tooling covers ~30-40% of WCAG; the
rest is manual. Not yet confirmed with a real screen reader (NVDA/VoiceOver).

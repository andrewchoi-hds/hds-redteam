# Changelog

All notable changes to hds-redteam are documented here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [1.1.0] — 2026-05-29

Reliability pass on the synthesis stage. The data contract between agents and
the dashboard is now fully specified, and the dashboard renders from a real
template instead of being regenerated from memory each run.

### Fixed
- **Template was unreachable at runtime.** The command referenced
  `examples/...` by a bare relative path, which does not resolve from a user's
  project directory once the plugin is installed. The command now reads
  `${CLAUDE_PLUGIN_ROOT}/templates/dashboard.html` explicitly.
- **Dashboard CSS was regenerated every run** (token-heavy, design drift). CSS
  is now extracted into a stable template; the command reuses the `<style>`
  block verbatim and only fills `{{TOKEN}}` slots.

### Added
- `templates/dashboard.html` — the canonical dashboard template with an
  embedded DATA CONTRACT and fill guide.
- **Likelihood and Impact** axes in every agent's output format. The risk
  matrix is now grounded in these two reported axes instead of being inferred
  from a single severity value.
- **Fixed confidence-to-percentage mapping** (HIGH=90, MEDIUM=65, LOW=35), so
  the confidence bars are deterministic rather than improvised.
- **Explicit verdict thresholds** (≥1 Critical → blocking; 0 Critical & ≥3
  High → conditional; otherwise pass-ish), with per-mode status labels and
  accent colors.
- **Blocker cards for High findings**, not just Critical. Previously a document
  with only High findings produced an empty blocker section.
- **Chrome localization** — all dashboard labels, headings, and axis text are
  now translated into the document's language to match the findings.
- **Large-document handling** — for docs over ~1,500 lines / 40KB, agents read
  the file by path instead of receiving the full text inline (avoids 4×
  token duplication).

## [1.0.0] — 2026-05-28

Initial release.

### Added
- Four parallel red-team agents: logic, technical, UX, business.
- `/redteam` command with five domain modes: `-prd`, `-sec`, `-rfc`, `-biz`,
  and default.
- Single self-contained HTML dashboard output with concrete scenarios,
  external citations, confidence scores, falsification conditions, risk
  matrix, timeline, and next actions.
- Example: `examples/housing-prd-review.html`.

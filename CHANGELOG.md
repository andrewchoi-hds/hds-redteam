# Changelog

All notable changes to hds-redteam are documented here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [1.3.2] — 2026-10-20

### Fixed
- Risk-matrix cell labels (`LOW · HIGH`, `HIGH · HIGH`, …) were overlapped by the
  finding dots. `.matrix-cell` filled from the top-left while the label sat
  absolutely positioned at the top-right, so any cell holding more than one row
  of dots covered its own axis label. The cell now reserves the label's height
  (`padding: 28px 12px 12px`). Surfaced only on a dense real document — the
  shipped demos top out at a handful of dots per cell, so it stayed invisible
  through four releases.

### Changed
- All four example dashboards re-synced with the fixed template CSS.

## [1.3.1] — 2026-10-20

### Fixed
- `/redteam` named its four agents by bare name (`redteam-logic`, …), which does
  not resolve for anyone who installed the plugin — agents provided by a plugin
  are namespaced (`hds-redteam:redteam-logic`). The command now names them
  exactly. This was broken for every external install since 1.1.0; it stayed
  hidden locally because same-named agents happened to exist in the project's
  own `.claude/agents/` directory and shadowed the problem.

## [1.3.0] — 2026-10-20

### Added
- `/draft` — writes a PRD draft (개요 / 유저 스토리 / 스토리별 요구사항) from a
  one-line brief or reference files, as the authoring counterpart to `/redteam`.
  It will not invent numbers: timers, limits and policies it cannot know go into
  a **미정** list, and anything inferred is tagged `[가정]`, so the draft hands
  `/redteam -prd` a concrete list of things to challenge.
  Deliberately uses no subagents — parallel authors contradict each other, and
  the four critic agents are hard-wired to report weaknesses only.

## [1.2.1] — 2026-08-24

### Fixed
- `commands/redteam.md` frontmatter failed to parse as YAML: the
  `argument-hint` value began with `[`, which YAML reads as a flow sequence,
  so the whole block was invalid and **every frontmatter field was silently
  dropped at runtime** (description, argument-hint). The value is now quoted.
  Caught by `claude plugin tag`'s deep validation, which checks per-file
  frontmatter that `claude plugin validate` does not.

### Added
- `description` field in `.claude-plugin/marketplace.json` (clears a validator
  warning and describes the marketplace in listings).

## [1.2.0] — 2026-05-30

### Added
- One rendered demo per domain mode, each generated end-to-end by the installed
  plugin (sample input `.md` + rendered dashboard `.html`):
  - `-sec` — `housing-system-design` (DO NOT DEPLOY, 7 Critical)
  - `-rfc` — `housing-realtime-inventory-rfc` (NEEDS REVISION, 5 Critical)
  - `-biz` — `housing-business-plan` (DO NOT LAUNCH, 5 Critical)
  (`-prd` demo `housing-prd-review.html` shipped in 1.0.0.)

### Changed
- README: all four mode demos in one table; Models section now names the
  frontmatter `model:` aliases (`opus`/`sonnet`) instead of pinned version
  numbers that would age; trimmed stale `-prd` demo figures.

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

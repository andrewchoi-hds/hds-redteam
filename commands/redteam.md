---
description: Run a 4-agent red-team review on a PRD and produce a single HTML dashboard with concrete scenarios, external citations, confidence scores, and falsification conditions.
argument-hint: <PRD file path or pasted content>
---

# Red Team Review

Critically review the following PRD by orchestrating four parallel red-team agents and producing a single HTML dashboard.

**Target:** $ARGUMENTS

## Procedure

### 1. Acquire input
- If `$ARGUMENTS` is a file path, Read it.
- If content is pasted, use it directly.
- If neither is available, ask the user for a PRD path or content and stop.
- Match the response language to the PRD's language (Korean PRD → Korean review, English PRD → English review).

### 2. Launch 4 red-team agents in parallel
Send all four Agent calls in **the same message** (parallel execution is mandatory):

- `redteam-logic` — logical consistency
- `redteam-technical` — technical feasibility
- `redteam-ux` — user and edge cases
- `redteam-business` — business and risk

For each agent, pass either the full PRD content in the prompt **or** the file path together with explicit instruction to Read it. Each agent must critique only from its own perspective.

### 3. Generate HTML dashboard
After all four agents return, synthesize their findings into a single, self-contained HTML file. Use `examples/housing-prd-review.html` (in this plugin) as the structural reference.

**Required sections (in order):**

1. **Verdict header** — GO / Conditional / NO-GO badge, tagline, meta strip (date, total findings, breakdown by severity, perspectives covered)
2. **Risk Matrix** — 3×3 grid (Likelihood × Impact). Place every finding as a dot inside the appropriate cell. Use color coding by severity. Tooltips on hover.
3. **Blocker cards** — one per Critical finding. Each card includes:
   - Number, severity tag, perspectives that raised it (visual indicator), confidence bar with percentage
   - **Scenario** — step-by-step concrete user / operator / system flow with names, times, amounts
   - **External evidence** — minimum 2 citations with source labels (law article numbers, paper titles, blog URLs, statistics with sources)
   - **Falsification condition** — explicit dismissal criterion
   - **Closing path** — who / what / estimated weeks
4. **Timeline** — Gantt-style bars for closing each blocker
5. **Next actions** — by role (decision-maker, PRD author, BD / engineering / legal)
6. **Footer** — meta info, source file path

**Visual style:**
- Dark mode (charcoal background, light text)
- Critical = red, High = orange, Medium = yellow, Low = gray
- System UI fonts, mono for codes / IDs
- Single self-contained HTML — no external dependencies (inline CSS, no CDN)

**Merging rule:** If multiple agents flagged the same issue from different angles, **merge them into one card** and tag with "raised by N perspectives". Severity follows the highest reported. Confidence is the average, with reasoning combined.

**Synthesis tone:** Don't sugar-coat. If no findings, say so honestly. Mark confidence openly — LOW confidence findings are still shown but explicitly marked LOW with a one-line reason.

### 4. Save output
Save the HTML file next to the input PRD:
- Input `path/to/prd.md` → Output `path/to/prd-redteam-review.html`
- If content was pasted, save to the current working directory as `redteam-review-<timestamp>.html`

### 5. Brief inline summary
After saving, output a short text summary (under 200 words) to the user containing:
- One-line verdict (GO / Conditional / NO-GO + count of blockers)
- Top 3 blockers (headline only)
- Path to the full dashboard
- Suggestion to open with `open <path>` (macOS) or equivalent

The HTML dashboard is the primary deliverable. The inline summary is just a pointer.

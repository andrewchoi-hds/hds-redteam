---
description: Multi-agent red-team review for proposal-style documents. Supports domain modes (-prd, -sec, -rfc, -biz) tailoring four parallel critics to each domain. Outputs a single HTML dashboard.
argument-hint: [-prd|-sec|-rfc|-biz] <file path or pasted content>
---

# Red Team Review

Critically review the target document by orchestrating four parallel red-team agents and producing a single self-contained HTML dashboard.

**Target:** $ARGUMENTS

## Procedure

### 1. Parse mode and acquire input

Parse `$ARGUMENTS` for an optional leading mode flag, then the document.

| Flag | Mode |
|---|---|
| `-prd` | Product Requirements Document |
| `-sec` | Security audit / threat review |
| `-rfc` | Technical RFC / architecture proposal |
| `-biz` | Business plan / GTM strategy |
| (none) | General domain-neutral critique |

The remaining argument is a file path or pasted content.
- File path → Read it
- Pasted content → use directly
- Nothing → ask user and stop

Match response language to document language (Korean doc → Korean review).

### 2. Build domain context block

Based on the parsed mode, prepare a context block to inject into every agent prompt:

**`-prd`**:
> This is a PRD (Product Requirements Document) review. Focus on: release readiness, Acceptance Criteria verifiability, scope clarity (V1 vs V2+ boundaries), KPIs, user flow completeness (happy path + error/edge), external dependencies, cross-team handoffs. Vocabulary: "release blocker", "policy variable", "must-answer-before-approval".

**`-sec`**:
> This is a security review. Focus on: threat model completeness (STRIDE-style), attack surface and trust boundaries, authentication/authorization flows, data protection (PII, secrets, encryption), input validation and injection vectors, operational security (logging, alerting, incident response), compliance (OWASP ASVS, ISO 27001, SOC2, GDPR, Korean PIPA), supply chain and third-party risks. Vocabulary: "deployment blocker", "vulnerability class", "CVSS-relevant exposure".

**`-rfc`**:
> This is a Technical RFC / architecture proposal review. Focus on: scalability (horizontal/vertical limits, hotspots), operability (deployability, observability, debuggability, on-call burden), blast radius (failure isolation, dependency cascades), rollback strategy and backward compatibility, performance budgets (latency, throughput, cost), alternatives considered and rejection rationale, migration plan from current state, distributed-systems concerns (consistency, idempotency, retry). Vocabulary: "design blocker", "operational risk", "architectural debt".

**`-biz`**:
> This is a business plan / GTM strategy review. Focus on: market fit (target segment, demand evidence, willingness to pay), unit economics (CAC, LTV, gross margin, payback period), competitive defensibility (moat, alternatives users have today), GTM execution (channel viability, sales cycle, distribution), regulatory and legal exposure, operational scalability (ops cost vs revenue scaling), financial assumptions and break-even, capital efficiency. Vocabulary: "launch blocker", "execution gap", "business risk".

**(no flag)**:
> This is a proposal-style document review. Apply your perspective broadly: look for missing assumptions, ambiguous claims, untested premises, and gaps in reasoning, feasibility, audience experience, or risk exposure.

### 3. Launch 4 red-team agents in parallel

Send all four Agent calls in **the same message** (parallel execution mandatory):

- `redteam-logic` — logical consistency
- `redteam-technical` — technical feasibility
- `redteam-ux` — user / audience perspective
- `redteam-business` — business / risk

For each agent, pass:
1. The **domain context block** from step 2 (paste the block at the top of the prompt)
2. The full document content (or file path with explicit Read instruction)
3. Reminder: critique only from your own perspective, but through the lens of the domain context

### 4. Generate HTML dashboard

Synthesize findings into a single self-contained HTML file. Use `examples/housing-prd-review.html` as the structural reference.

Required sections (in order):
1. **Verdict header** — verdict badge, tagline, meta strip (date, mode, total findings, severity breakdown, perspectives covered)
2. **Risk Matrix** — 3×3 grid (Likelihood × Impact), every finding placed as a dot inside the appropriate cell; tooltips on hover; color-coded by severity
3. **Blocker cards** — one card per Critical finding, each with:
   - Number, severity tag, perspectives raising it (visual indicator), confidence bar with percentage
   - **Scenario** — step-by-step concrete actor/system flow with names, times, amounts
   - **External evidence** — minimum 2 citations with source labels (law article numbers, paper titles, blog URLs, statistics with sources)
   - **Falsification condition** — explicit dismissal criterion
   - **Closing path** — who / what / estimated weeks
4. **Timeline** — Gantt-style bars for closing each blocker
5. **Next actions** by role
6. **Footer** with meta info and source path

Adapt verdict label and vocabulary to the mode:
- `-prd` → "NO-GO" / "Release blockers"
- `-sec` → "DO NOT DEPLOY" / "Critical vulnerabilities"
- `-rfc` → "NEEDS REVISION" / "Design blockers"
- `-biz` → "DO NOT LAUNCH" / "Execution blockers"
- (default) → "NEEDS WORK" / "Blocking issues"

Visual style: dark mode, single self-contained HTML, no external dependencies (inline CSS, no CDN).

Merging rule: if multiple agents flag the same issue from different angles, **merge them into one card** and tag with "raised by N perspectives". Severity follows the highest reported.

Tone: do not sugar-coat. If no findings, say so honestly. Show LOW confidence findings openly with their reason.

### 5. Save output

Save next to the input:
- File `path/to/doc.md` → `path/to/doc-redteam-<mode>.html` (omit `-<mode>` if default)
- Pasted content → `redteam-review-<mode>-<timestamp>.html` in the current working directory

### 6. Brief inline summary

After saving, output a short (<200 words) inline summary:
- Mode used
- One-line verdict
- Critical count
- Top 3 blockers (headline only)
- Path to the full dashboard
- macOS open hint: `open <path>`

The HTML dashboard is the primary deliverable. The inline summary is just a pointer.

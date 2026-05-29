---
description: Multi-agent red-team review for proposal-style documents. Supports domain modes (-prd, -sec, -rfc, -biz) tailoring four parallel critics to each domain. Outputs a single HTML dashboard.
argument-hint: [-prd|-sec|-rfc|-biz] <file path or pasted content>
---

# Red Team Review

Critically review the target document by orchestrating four parallel red-team agents and rendering their findings into a single self-contained HTML dashboard.

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
- File path → Read it. **If the document is large (>1,500 lines or >40KB), do NOT paste its full text into agent prompts** — instead pass the absolute file path and instruct each agent to Read it themselves (they have the Read tool). This avoids 4× token duplication.
- Pasted content → use directly.
- A mode flag with no document, or nothing at all → ask the user for a path/content and stop.

Detect the document's primary language now; you will localize the dashboard chrome to it later.

### 2. Build domain context block

Based on the parsed mode, prepare a context block to inject verbatim at the top of every agent prompt:

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
1. The **domain context block** from step 2
2. The document content, OR (for large docs) the absolute file path with an explicit Read instruction
3. Reminder: critique only from your own perspective, through the lens of the domain context. Every finding MUST include Severity, **Likelihood**, **Impact**, Confidence, Scenario, External evidence, Falsification, Closing path.

### 4. Render the HTML dashboard from the template

**Read the template first:** `${CLAUDE_PLUGIN_ROOT}/templates/dashboard.html`. Keep its `<style>` block **verbatim** — do not regenerate CSS. Replace every `{{TOKEN}}` and the marked body regions with synthesized content. (For a fully worked reference of what filled output looks like, see `${CLAUDE_PLUGIN_ROOT}/examples/housing-prd-review.html`.)

Follow the DATA CONTRACT in the template's header comment. Key rules restated:

- **Confidence → percentage** (fixed mapping, no improvisation): `HIGH = 90`, `MEDIUM = 65`, `LOW = 35`. When merging duplicates, average the numeric values.
- **Risk Matrix axes** come from each finding's **Likelihood** and **Impact** lines — NOT from severity. Severity only sets the dot color (`crit`/`high`/`med`/`low`).
- **Blocker cards** are generated for every **Critical AND High** finding. Number them `C1..Cn` then `H1..Hn` and reuse those exact IDs as the matrix dot labels. Medium/Low appear only as matrix dots.
- **Verdict rule** (status text + `--verdict` accent color):
  - `≥1 Critical` → blocking verdict, accent `--critical`
  - `0 Critical & ≥3 High` → conditional verdict, accent `--high`
  - `0 Critical & <3 High` → pass-ish verdict, accent `--ok`
  - Status label by mode: `-prd` NO-GO/CONDITIONAL/GO · `-sec` DO NOT DEPLOY/CONDITIONAL/CLEAR · `-rfc` NEEDS REVISION/MINOR REVISION/APPROVED · `-biz` DO NOT LAUNCH/CONDITIONAL/GO · none NEEDS WORK/MINOR/OK
- **Localization:** translate ALL `{{LABEL_*}}` tokens, section headings, row labels, axis labels, and toggle text into the document's language. The agents' findings are already in that language; the chrome must match so the dashboard is not half-English.
- **Merging rule:** if multiple agents flag the same issue, merge into one card, tag "raised by N perspectives", light up those perspective dots, take the highest severity and the most alarming Likelihood/Impact.
- **Tone:** do not sugar-coat. Show LOW-confidence findings openly, marked LOW. If genuinely nothing critical, say so in the verdict honestly.

### 5. Save output

Save next to the input:
- File `path/to/doc.md` → `path/to/doc-redteam-<mode>.html` (omit `-<mode>` if default mode)
- Pasted content → `redteam-review-<mode>.html` in the current working directory (append a short disambiguating suffix if it already exists)

### 6. Brief inline summary

After saving, print a short (<200 words) inline summary:
- Mode used
- One-line verdict (label + Critical/High counts)
- Top 3 blockers (headline only)
- Absolute path to the dashboard
- Open hint: `open <path>` (macOS) / `xdg-open <path>` (Linux) / `start <path>` (Windows)

The HTML dashboard is the primary deliverable. The inline summary is just a pointer.

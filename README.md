# hds-redteam

> Multi-agent red-team review for proposal-style documents. Four parallel critics + five domain modes produce a single HTML dashboard grounded in concrete scenarios, external citations, confidence scores, and falsification conditions.

**[Live demo](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-prd-review.html)** · [Install](#install) · [Usage](#usage) · [Modes](#modes) · [How it works](#how-it-works)

---

## What it does

When you have a proposal-style document that needs critical review before commitment, `/redteam` launches four parallel critic agents — each looking at the document through one focused angle:

- 🧠 **Logic** — contradictions, missing premises, undefined terms, unverifiable claims
- ⚙️ **Technical** — hidden complexity, dependencies, concurrency, observability
- 👤 **UX / Audience** — missing scenarios, error flows, accessibility, trust violations
- 💼 **Business / Risk** — side effects, regulation, ops cost, measurability, legal risk

A **mode flag** tailors all four to a specific domain. Their findings are synthesized into a **single self-contained HTML dashboard** featuring:

- 🎯 **Concrete scenarios** — every finding includes a step-by-step story (names, times, amounts) showing how the gap fails in practice
- 📚 **External citations** — laws with article numbers, standards (ISO/IEC/IEEE 29148, CAP theorem, WCAG, OWASP), industry incidents (Airbnb, Toss, Stripe), and statistics
- 📊 **Confidence scores** — every finding labeled HIGH / MEDIUM / LOW with a one-line reason
- ✓ **Falsification conditions** — each finding states the evidence that would dismiss it
- 📅 **Closing path** — who fixes what, in how long
- 🎨 **Risk matrix** — every finding mapped on a 3×3 likelihood × impact grid

## Install

In Claude Code:

```
/plugin marketplace add anthropics/claude-plugins-community
/plugin install hds-redteam@claude-community
```

For local development:

```
git clone https://github.com/andrewchoi-hds/hds-redteam.git
```
Then in Claude Code:
```
/plugin marketplace add /path/to/hds-redteam
/plugin install hds-redteam
```

## Usage

```
/redteam [mode] <file path or pasted content>
```

Examples:

```
/redteam -prd path/to/prd.md                  # PRD review
/redteam -sec path/to/system-design.md        # Security audit
/redteam -rfc path/to/architecture-rfc.md     # Technical RFC critique
/redteam -biz path/to/business-plan.md        # Business / GTM stress-test
/redteam path/to/any-proposal.md              # General (no mode)
```

The command:

1. Parses the mode flag (if any)
2. Reads your document
3. Launches all four red-team agents in parallel, each with the domain context for the selected mode
4. Synthesizes their findings into a single self-contained HTML file
5. Saves alongside your document: `your-doc-redteam-<mode>.html`
6. Prints a short inline summary with the verdict and top blockers

Open the HTML in any browser. No build step. No external dependencies.

## Modes

Same 4 critics. Different domain lens.

| Mode | Domain | Each critic focuses on |
|---|---|---|
| `-prd` | Product Requirements Document | Release readiness, AC verifiability, scope (V1/V2 boundaries), KPIs, user flow completeness, cross-team handoffs |
| `-sec` | Security review / threat audit | Threat model, attack surface, auth flows, data protection, opsec, compliance (OWASP, ISO 27001, GDPR, PIPA) |
| `-rfc` | Technical RFC / architecture | Scalability, operability, blast radius, rollback, alternatives considered, distributed-systems concerns |
| `-biz` | Business plan / GTM strategy | Market fit, unit economics (CAC/LTV/margin), defensibility, GTM execution, regulatory exposure |
| (none) | General domain-neutral critique | Missing assumptions, ambiguous claims, untested premises, gaps in reasoning / feasibility / audience / risk |

The verdict label adapts to mode:

| Mode | Verdict label | Blocker label |
|---|---|---|
| `-prd` | NO-GO | Release blockers |
| `-sec` | DO NOT DEPLOY | Critical vulnerabilities |
| `-rfc` | NEEDS REVISION | Design blockers |
| `-biz` | DO NOT LAUNCH | Execution blockers |
| (default) | NEEDS WORK | Blocking issues |

## Example output

See [examples/housing-prd-review.html](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-prd-review.html) for a `-prd` mode review of a sample Housing Marketplace V1 PRD.

What you'll see in the demo:

- **NO-GO verdict** with 5 release-blocking items
- **3×3 risk matrix** placing all 30+ findings
- Each blocker card includes:
  - Step-by-step scenario with hypothetical user names and amounts
  - Real legal citations (Korean real-estate licensing law article numbers + penalties)
  - Distributed-systems references (CAP theorem, Saga pattern, 2PC limits)
  - Industry parallels (Airbnb inventory sync, Toss payment SLAs)
  - Confidence bars (75%–95%) with one-line reasoning
  - Falsification conditions ("this finding is dismissed if a legal opinion exists")
  - Closing path with estimated weeks

Demos for `-sec`, `-rfc`, `-biz` modes will be added in V1.1.

## How it works

### Why multi-agent?

A single critic prompt drifts toward the most obvious problems and softens its tone over a long response. Splitting into four perspectives — each with its own system prompt, reference base, and model — produces sharper findings. The synthesis step then flags issues raised from multiple angles as higher priority.

### Why mode flags?

The four perspectives are domain-neutral by design. But a PRD reviewer and a security auditor look at "logic" or "risk" through different lenses. A mode flag injects a domain context block into every agent's prompt, so the same critic shifts its vocabulary and focus to fit the domain.

This keeps the agent set small (4 agents) while supporting many domains.

### Models

- **Logic & Technical**: Claude Opus 4.7 — deep reasoning where gaps are hardest to spot
- **UX & Business**: Claude Sonnet 4.6 — broad enumeration of scenarios and risks

Override in each agent's frontmatter to change the mix.

### Why an HTML dashboard?

Earlier iterations tried text reports, condensed checklists, and 1-page summaries. They failed in different ways:

- **Text reports** are too long to skim
- **Condensed checklists** hide the gravity of issues
- **1-page summaries** drop the evidence, making readers question the analysis

The HTML dashboard combines three things in one artifact:

- **Impact** — concrete step-by-step scenarios make consequences visceral
- **Clarity** — visual risk matrix lets you see all findings at once
- **Trust** — external citations and confidence scores let readers calibrate

## Language

System prompts are in English, but each agent responds in the language of the input document. Korean document → Korean review. English document → English review.

## Project structure

```
hds-redteam/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── redteam-logic.md
│   ├── redteam-technical.md
│   ├── redteam-ux.md
│   └── redteam-business.md
├── commands/
│   └── redteam.md
├── examples/
│   └── housing-prd-review.html
├── README.md
└── LICENSE
```

## Contributing

Issues and PRs welcome. Particularly interested in:

- Additional modes (e.g., `-mkt` for marketing strategy, `-design` for UX design proposals, `-okr` for OKR documents, `-paper` for academic submissions, `-policy` for organizational policies)
- Reference bases for non-Korean / non-US regulatory contexts (EU, SEA, LATAM)
- Additional perspectives beyond the four core agents
- Improvements to the synthesis prompt and dashboard layout

## License

MIT — see [LICENSE](LICENSE).

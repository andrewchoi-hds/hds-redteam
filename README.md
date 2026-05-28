# hds-redteam

> Multi-agent red team review for PRDs. Four parallel critics produce a single HTML dashboard grounded in concrete scenarios, external citations, confidence scores, and falsification conditions.

**[Live demo](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-prd-review.html)** · [Install](#install) · [Usage](#usage) · [How it works](#how-it-works)

---

## What it does

When you have a PRD that needs critical review before approval, `/redteam` launches four parallel critic agents — each looking at the document through a single, focused angle:

- 🧠 **Logic** — contradictions, missing premises, undefined terms, unverifiable claims
- ⚙️ **Technical** — hidden complexity, dependencies, concurrency, observability
- 👤 **UX** — missing scenarios, error flows, accessibility, trust violations
- 💼 **Business** — side effects, regulation, ops cost, measurability, legal risk

Their findings are synthesized into a **single self-contained HTML dashboard** featuring:

- 🎯 **Concrete scenarios** — every finding includes a step-by-step story (hypothetical names, times, amounts) showing how the gap fails in production
- 📚 **External citations** — laws and article numbers, standards (ISO/IEC/IEEE 29148, CAP theorem, WCAG), industry incidents (Airbnb double-booking, Toss / Stripe outages), and statistics
- 📊 **Confidence scores** — every finding labeled HIGH / MEDIUM / LOW with a one-line reason, so you can tell certainty from hunch
- ✓ **Falsification conditions** — each finding states the evidence that would dismiss it, giving the PRD author defense room
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
/redteam path/to/your-prd.md
```

Or paste content directly:

```
/redteam
> (paste your PRD content)
```

The command:

1. Reads your PRD
2. Launches all four red-team agents in parallel
3. Synthesizes their findings into a single self-contained HTML file
4. Saves it alongside your PRD as `your-prd-redteam-review.html`
5. Prints a short inline summary with the verdict and top blockers

Open the HTML file in any browser. No build step, no external dependencies.

## Example output

See [examples/housing-prd-review.html](examples/housing-prd-review.html) for a full review of a sample Housing Marketplace V1 PRD.

What you'll see in the demo:

- **NO-GO verdict** with 5 release-blocking items
- **3×3 risk matrix** placing all 30+ findings
- Each blocker card containing:
  - A step-by-step scenario (with hypothetical user names and amounts)
  - Real legal citations (Korean real-estate licensing law article numbers + penalties)
  - Distributed-systems references (CAP theorem, Saga pattern, 2PC limits)
  - Industry parallels (Airbnb inventory sync, Toss payment SLAs)
  - Confidence bars (75%–95%) with one-line reasoning
  - Falsification conditions ("this is dismissed if a legal opinion exists")
  - Closing path with estimated weeks

## How it works

### Why multi-agent?

A single critic prompt drifts toward the most obvious problems and softens its tone over a long response. Splitting into four perspectives — each with its own system prompt, reference base, and model — produces sharper findings. The synthesis step then flags issues raised from multiple angles as higher priority.

### Models

- **Logic & Technical**: Claude Opus 4.7 — deep reasoning where the gaps are hardest to spot
- **UX & Business**: Claude Sonnet 4.6 — broad enumeration of scenarios and risks

Override in each agent's frontmatter if you want a different mix.

### Why an HTML dashboard?

Earlier iterations tried text reports, condensed checklists, and 1-page summaries. They all failed in different ways:

- **Text reports** are too long to skim. Readers stop after the first three findings.
- **Condensed checklists** hide the gravity of issues. "Policy variables missing" reads no heavier than "page numbering missing".
- **1-page summaries** drop the evidence, making readers question the analysis.

The HTML dashboard solved this by combining three things in one artifact:
- **Impact** — concrete step-by-step scenarios make consequences visceral
- **Clarity** — visual risk matrix lets you see all findings at once
- **Trust** — external citations and confidence scores let readers calibrate

## Language

System prompts are written in English, but each agent responds in the language of the input PRD. Korean PRD → Korean review. English PRD → English review.

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

- Reference bases for non-Korean / non-US regulatory contexts (EU, SEA, LATAM)
- Additional perspectives beyond the four core agents (security, accessibility, GTM)
- Improvements to the synthesis prompt and dashboard layout

## License

MIT — see [LICENSE](LICENSE).

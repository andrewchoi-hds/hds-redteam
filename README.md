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

## Team / organization deployment

The GitHub repo *is* the marketplace — nobody needs to clone it by hand.

**Each teammate, one time:**

```
/plugin marketplace add andrewchoi-hds/hds-redteam
/plugin install hds-redteam@hds-redteam-marketplace
```

**Or auto-enable it for everyone who opens a shared repo** — commit this to that
repo's `.claude/settings.json` and teammates get the plugin on trust-prompt
accept, with no commands to run:

```json
{
  "extraKnownMarketplaces": {
    "hds-redteam-marketplace": {
      "source": { "source": "github", "repo": "andrewchoi-hds/hds-redteam" }
    }
  },
  "enabledPlugins": { "hds-redteam@hds-redteam-marketplace": true }
}
```

**Org-wide (admin):** the same two keys work in managed settings, which applies
to every user in the organization. See
[managed settings](https://code.claude.com/docs/en/managed-settings.md) and
[plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces.md).

> Adding the marketplace from a **local directory** instead (`/plugin
> marketplace add /path/to/hds-redteam`) only works on your own machine — handy
> for developing the plugin, useless for sharing it.

## Updating

Installed plugins are **cached** — a new release on GitHub does not update your
local copy automatically. After a new version ships, refresh the marketplace
and reinstall:

```
/plugin marketplace update hds-redteam-marketplace
/plugin install hds-redteam@hds-redteam-marketplace
```

(For the official community marketplace, use `@claude-community` instead.)

To confirm which version is active, check `version` in the plugin's
`.claude-plugin/plugin.json`, or run `/plugin` and inspect the installed entry.
If `/redteam` still references old behavior after updating, fully remove and
re-add the marketplace:

```
/plugin marketplace remove hds-redteam-marketplace
/plugin marketplace add andrewchoi-hds/hds-redteam
/plugin install hds-redteam@hds-redteam-marketplace
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

One rendered demo per mode — each generated end-to-end by the installed plugin, reviewing a sample document from the same fictional "Housing" product:

| Mode | Input | Dashboard | Verdict |
|---|---|---|---|
| `-prd` | Housing V1 PRD | [housing-prd-review.html](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-prd-review.html) | NO-GO |
| `-sec` | [housing-system-design.md](examples/housing-system-design.md) | [housing-system-design-redteam-sec.html](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-system-design-redteam-sec.html) | DO NOT DEPLOY |
| `-rfc` | [housing-realtime-inventory-rfc.md](examples/housing-realtime-inventory-rfc.md) | [housing-realtime-inventory-rfc-redteam-rfc.html](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-realtime-inventory-rfc-redteam-rfc.html) | NEEDS REVISION |
| `-biz` | [housing-business-plan.md](examples/housing-business-plan.md) | [housing-business-plan-redteam-biz.html](https://andrewchoi-hds.github.io/hds-redteam/examples/housing-business-plan-redteam-biz.html) | DO NOT LAUNCH |

Each blocker card in a dashboard carries:

- A step-by-step **scenario** with hypothetical actors, times, and amounts
- **External citations** — law article numbers, standards (ISO/IEC/IEEE 29148, CAP theorem, OWASP/CWE, WCAG), and industry incidents
- A **confidence bar** with a one-line reason
- A **falsification condition** — the evidence that would dismiss the finding
- A **closing path** — who fixes it, and in how long

Findings are placed on a 3×3 likelihood × impact **risk matrix**, and each blocker gets a Gantt-style closing **timeline**.

## How it works

### Why multi-agent?

A single critic prompt drifts toward the most obvious problems and softens its tone over a long response. Splitting into four perspectives — each with its own system prompt, reference base, and model — produces sharper findings. The synthesis step then flags issues raised from multiple angles as higher priority.

### Why mode flags?

The four perspectives are domain-neutral by design. But a PRD reviewer and a security auditor look at "logic" or "risk" through different lenses. A mode flag injects a domain context block into every agent's prompt, so the same critic shifts its vocabulary and focus to fit the domain.

This keeps the agent set small (4 agents) while supporting many domains.

### Models

Each agent pins a model **tier** in its frontmatter (`model:` field), not a
version. Tier aliases always resolve to the newest model in that tier, so the
plugin picks up model upgrades with no edit required (as of 2026-05: Opus 5 and
Sonnet 5).

- **Logic & Technical**: `opus` — deep reasoning where gaps are hardest to spot
- **UX & Business**: `sonnet` — broad enumeration of scenarios and risks

Change the `model:` alias in an agent's frontmatter to adjust the mix. Pinning a
full model ID (e.g. `claude-opus-5`) also works, but then upgrades become manual
and the agent breaks for anyone without access to that exact model.

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
├── templates/
│   └── dashboard.html        # canonical dashboard template + data contract
├── examples/                 # sample inputs + one rendered dashboard per mode
│   ├── housing-prd-review.html
│   ├── housing-system-design.md          + …-redteam-sec.html
│   ├── housing-realtime-inventory-rfc.md + …-redteam-rfc.html
│   └── housing-business-plan.md          + …-redteam-biz.html
├── CHANGELOG.md
├── README.md
└── LICENSE
```

## Contributing

Issues and PRs welcome. Particularly interested in:

- Additional modes (e.g., `-mkt` for marketing strategy, `-design` for UX design proposals, `-okr` for OKR documents, `-paper` for academic submissions, `-policy` for organizational policies)
- Reference bases for non-Korean / non-US regulatory contexts (EU, SEA, LATAM)
- Additional perspectives beyond the four core agents
- Improvements to the synthesis prompt and dashboard layout

## Privacy

This plugin runs entirely in your local Claude Code environment using your own Anthropic API credentials. The maintainers do **not** collect, transmit, or store any data related to your documents or usage:

- Your documents are sent directly from your machine to Anthropic's API. They never touch any server operated by the plugin author.
- The plugin author has no access to your API key, your prompts, the agents' responses, or the generated HTML reports.
- The plugin has no telemetry, analytics, or third-party tracking.
- All generated review files are saved to your local filesystem only.

If you fork, modify, or redistribute this plugin, please preserve this property or update this section to reflect any changes.

## License

MIT — see [LICENSE](LICENSE).

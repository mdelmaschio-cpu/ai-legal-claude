# ai-legal-claude

AI-powered contract review and legal document generation suite for Claude Code. 14 skills + 5 parallel agents for comprehensive contract analysis and document generation. Designed for freelancers and small businesses.

## Repository Purpose

Provides the `/legal` command family in Claude Code for contract review, risk analysis, document generation, and PDF reporting. The flagship feature (`/legal review`) orchestrates 5 parallel agents for comprehensive contract analysis with a Contract Safety Score.

## Structure

```
ai-legal-claude/
├── README.md
├── install.sh                 # One-line installer (executable)
├── uninstall.sh               # Cleanup script (executable)
├── generate_sample_contract.py  # ReportLab contract generator for testing
├── sample-contract.pdf        # Pre-generated test document
├── assets/
│   └── banner.svg
├── legal/
│   └── SKILL.md               # Orchestrator: routes /legal <verb> to skills
├── skills/                    # 14 individual skill directories
│   ├── legal-review/          # Flagship: 5-agent full contract review
│   ├── legal-risks/           # Deep risk analysis
│   ├── legal-compare/         # Side-by-side contract comparison
│   ├── legal-plain/           # Legalese → plain English
│   ├── legal-negotiate/       # Counter-proposal generator
│   ├── legal-missing/         # Missing protections finder
│   ├── legal-nda/             # NDA generator
│   ├── legal-terms/           # Terms of service generator
│   ├── legal-privacy/         # Privacy policy generator
│   ├── legal-agreement/       # Business agreement templates
│   ├── legal-compliance/      # Compliance gap analysis
│   ├── legal-freelancer/      # Freelancer contract review
│   └── legal-report-pdf/      # PDF report generator (ReportLab)
├── agents/                    # 5 parallel agent profiles
│   ├── legal-clauses.md
│   ├── legal-risks.md
│   ├── legal-compliance.md
│   ├── legal-terms.md
│   └── legal-recommendations.md
├── scripts/
│   └── generate_legal_pdf.py  # ReportLab PDF generation
└── templates/
    └── contract-review-template.md
```

## Command Routing

All commands enter through `legal/SKILL.md`, which routes to the appropriate skill:

| Command | Skill | Description |
|---------|-------|-------------|
| `/legal review` | `legal-review/` | Full 5-agent analysis with Safety Score |
| `/legal risks` | `legal-risks/` | Deep risk analysis only |
| `/legal compare` | `legal-compare/` | Side-by-side comparison of two contracts |
| `/legal plain` | `legal-plain/` | Plain English translation |
| `/legal negotiate` | `legal-negotiate/` | Counter-proposals and redlines |
| `/legal missing` | `legal-missing/` | Missing protections finder |
| `/legal nda` | `legal-nda/` | Generate NDA |
| `/legal terms` | `legal-terms/` | Generate Terms of Service |
| `/legal privacy` | `legal-privacy/` | Generate Privacy Policy |
| `/legal agreement` | `legal-agreement/` | Generate business agreement |
| `/legal compliance` | `legal-compliance/` | Compliance gap analysis |
| `/legal freelancer` | `legal-freelancer/` | Freelancer-specific review |
| `/legal report` | `legal-report-pdf/` | Export PDF report |

## Flagship Feature: /legal review

Five agents run in parallel, each analyzing a different dimension:

| Agent | File | Focus |
|-------|------|-------|
| Clauses | `legal-clauses.md` | Clause-by-clause extraction and analysis |
| Risks | `legal-risks.md` | Risk categorization (High/Medium/Low) |
| Compliance | `legal-compliance.md` | GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2 |
| Terms | `legal-terms.md` | Key terms, definitions, obligations timeline |
| Recommendations | `legal-recommendations.md` | Negotiation priorities and fix suggestions |

Output includes:
- **Contract Safety Score** (0–100 with letter grade A–F)
- Risk dashboard with categorized findings
- Obligations timeline
- Negotiation priorities ranked by importance
- Specific redline suggestions with proposed language

## Contract Type Detection

The skills automatically classify contracts before analysis:

- Service Agreement / Statement of Work
- Non-Disclosure Agreement (NDA)
- SaaS / Master Service Agreement (MSA)
- Employment Agreement
- Independent Contractor Agreement
- Purchase Agreement / M&A
- Terms of Service / Privacy Policy

Detection happens upfront; type-specific checklists are applied automatically.

## Position-Aware Analysis

Skills accept a `--position` flag or infer position from context:
- `buyer` / `seller`
- `vendor` / `customer`
- `employer` / `employee`
- `licensor` / `licensee`

Risk levels shift based on position (e.g., an indemnification clause that's low risk for vendors may be high risk for customers).

## PDF Generation

`scripts/generate_legal_pdf.py` requires Python 3.8+ and ReportLab:

```bash
pip install reportlab
python3 scripts/generate_legal_pdf.py --input review-output.md --output report.pdf
```

`generate_sample_contract.py` generates test PDFs for development:

```bash
python3 generate_sample_contract.py  # writes sample-contract.pdf
```

## Installation

```bash
# Install
curl -fsSL https://raw.githubusercontent.com/mdelmaschio-cpu/ai-legal-claude/main/install.sh | bash

# Uninstall
~/.claude/skills/ai-legal-claude/uninstall.sh
```

`install.sh` clones the repo and copies skills to `~/.claude/skills/`. It requires Claude Code with an Anthropic API key.

## Development Workflow

1. Edit the relevant `skills/<skill-name>/SKILL.md`
2. If changing the command interface, update the routing table in `legal/SKILL.md`
3. Test with `sample-contract.pdf` (pre-generated) or generate a new one:
   ```bash
   python3 generate_sample_contract.py
   ```
4. Verify PDF generation if touching `scripts/generate_legal_pdf.py`:
   ```bash
   pip install reportlab && python3 scripts/generate_legal_pdf.py --help
   ```

No automated test suite. Manual testing with the sample contract is the standard verification method.

## Key Conventions

- **Always include legal disclaimer**: Every skill output must state it is not legal advice and recommend consulting a licensed attorney for significant decisions.
- **Jurisdiction awareness**: Default to US law; flag where analysis may differ by jurisdiction.
- **Structured output**: Use tables for risk dashboards and obligation timelines, not prose.
- **No hallucinated legal citations**: Only cite standards/regulations that are explicitly present in the document being reviewed.
- **Safety Score calibration**: A score of 70+ = reasonably balanced; 50–69 = significant concerns; below 50 = do not sign without legal review.
- **Parallel agents are the point**: The `/legal review` skill must use all 5 agents concurrently, never sequentially.

# AI Legal Claude

AI-powered contract review and legal document generation system for Claude Code. Reviews contracts, flags risks, generates legal documents, checks compliance, and produces PDF reports — all via `/legal` slash commands.

**Stats:** 14 skills · 5 parallel agents · Python PDF generation via ReportLab.

## Repository Structure

```
ai-legal-claude/
├── README.md                       # Full documentation, command reference, use cases
├── install.sh                      # One-line installer (copies skills/agents/scripts)
├── uninstall.sh                    # Clean removal script
├── generate_sample_contract.py     # Utility: generates sample PDFs for testing
├── sample-contract.pdf             # Example contract for testing
├── assets/                         # Banner SVG and visual assets
├── legal/
│   └── SKILL.md                    # Main orchestrator — routes all /legal commands
├── skills/
│   ├── legal-review/SKILL.md       # Flagship: full review via 5 parallel agents
│   ├── legal-risks/SKILL.md        # Deep risk analysis with financial exposure estimates
│   ├── legal-compare/SKILL.md      # Side-by-side comparison of two contract versions
│   ├── legal-plain/SKILL.md        # Plain-English translation of legalese
│   ├── legal-negotiate/SKILL.md    # Counter-proposal generator with replacement language
│   ├── legal-missing/SKILL.md      # Finds missing protections in a contract
│   ├── legal-nda/SKILL.md          # NDA generator (mutual/one-way/employee/vendor)
│   ├── legal-terms/SKILL.md        # Terms of service generator
│   ├── legal-privacy/SKILL.md      # Privacy policy generator
│   ├── legal-agreement/SKILL.md    # Business agreement generator (freelancer, SOW, MSA)
│   ├── legal-compliance/SKILL.md   # Compliance gap analysis (GDPR, CCPA, ADA, PCI-DSS)
│   ├── legal-freelancer/SKILL.md   # Freelancer-perspective contract review
│   └── legal-report-pdf/SKILL.md  # PDF report with score gauges and risk charts
├── agents/
│   ├── legal-clauses.md            # Clause analysis (identifies and categorizes)
│   ├── legal-risks.md              # Risk assessment (scores each clause)
│   ├── legal-compliance.md         # Compliance check (regulatory flags)
│   ├── legal-terms.md              # Terms & obligations (deadlines, triggers)
│   └── legal-recommendations.md   # Recommendations (specific fix language)
├── scripts/
│   └── generate_legal_pdf.py       # PDF generation — requires `pip3 install reportlab`
└── templates/
    └── contract-review-template.md # Structured report output template
```

## Command Reference

All commands use the `/legal` prefix, routed through `legal/SKILL.md`:

| Command | What It Does |
|---------|-------------|
| `/legal review <file>` | **Flagship** — full review, Contract Safety Score (0–100), clause analysis |
| `/legal risks <file>` | Deep risk analysis with financial exposure estimates |
| `/legal compare <file1> <file2>` | Side-by-side comparison — flags additions, removals, dangerous changes |
| `/legal plain <file>` | Translates every clause to plain English |
| `/legal negotiate <file>` | Counter-proposals with replacement language for unfavorable clauses |
| `/legal missing <file>` | Finds protections that should exist but don't |
| `/legal nda <description>` | Generates custom NDA |
| `/legal terms <url>` | Generates Terms of Service |
| `/legal privacy <url>` | Generates Privacy Policy |
| `/legal agreement <type>` | Generates business agreements (freelancer, partnership, SOW, MSA) |
| `/legal freelancer <file>` | Freelancer-perspective review |
| `/legal compliance <url>` | GDPR/CCPA/ADA/PCI-DSS/SOC2 gap analysis |
| `/legal report-pdf` | Professional PDF with score gauges and prioritized actions |

## The Flagship: `/legal review`

Launches 5 agents in parallel with weighted contributions:

| Agent | File | Weight | Role |
|-------|------|--------|------|
| Clause Analyst | `agents/legal-clauses.md` | 20% | Identifies and categorizes every clause |
| Risk Assessor | `agents/legal-risks.md` | 25% | Scores each clause for risk |
| Compliance Checker | `agents/legal-compliance.md` | 20% | Flags regulatory issues |
| Terms Mapper | `agents/legal-terms.md` | 15% | Maps obligations, deadlines, and triggers |
| Recommendations Engine | `agents/legal-recommendations.md` | 20% | Generates specific replacement language |

## Installation

```bash
# One-line install
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# Or run locally
./install.sh

# PDF generation dependency (optional)
pip3 install reportlab
```

## Development Conventions

- **Each skill is self-contained** — `skills/<name>/SKILL.md` contains all instructions for that command; no shared state between skills
- **The orchestrator (`legal/SKILL.md`) only routes** — it does not contain domain knowledge or analysis logic
- **Agent files are single-purpose** — each agent has exactly one scoring or analysis responsibility in the 5-agent `/legal review` flow
- **PDF generation is pure Python/ReportLab** — no external API calls, no network dependencies
- **New document types** follow this structure: Pre-Review Checklist → Position Identification → Output Format → Red Flags → Type-Specific Checklist

## Output Structure (standard for all review commands)

1. Document header (type, party, counterparty, risk level, status)
2. Pre-signing alerts (blank fields, missing exhibits)
3. Executive summary
4. Key terms table
5. Red flags quick scan
6. Risk analysis (🔴 Critical → 🟡 Important → 🟢 Acceptable)
7. Missing provisions with suggested language
8. Internal consistency issues
9. Negotiation priority list

## AI Assistant Guidelines

- **Disclaimer is mandatory** — every output from every skill must include "This tool is for informational purposes only and does not provide legal advice"
- **Position-awareness is required** — always ask which party the user represents if not stated; this changes what is considered "risky"
- **No hallucination** — only reference text that actually exists in the document under review
- **Show what is acceptable** — always include a "Reviewed & Acceptable" or equivalent section; do not only flag red flags
- **Market benchmarks** are defined in the skill files — use them consistently (e.g. 12-month liability cap = market standard)
- **Agent weights are calibrated** — do not change the percentage weights in the 5-agent review without testing the scoring impact
- **PDF generation requires ReportLab** — check for the dependency before calling `generate_legal_pdf.py`; surface a clear error if absent

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

**ai-legal-claude** is a Claude Code skills package for AI-powered contract review and legal document generation. It installs 14 skills, 5 specialist agents, and PDF generation scripts via a one-command installer.

There is no build system or test framework. All intelligence lives in Markdown skill files; the only runnable code is the Python PDF generator.

## Repository Layout

```
ai-legal-claude/
├── legal/
│   └── SKILL.md                    # Main router — dispatches to sub-skills
├── skills/
│   ├── legal-review/SKILL.md       # Flagship: full contract review (5 parallel agents)
│   ├── legal-risks/SKILL.md        # Deep risk analysis + financial exposure scoring
│   ├── legal-compare/SKILL.md      # Side-by-side contract version comparison
│   ├── legal-plain/SKILL.md        # Legalese → plain English translation
│   ├── legal-negotiate/SKILL.md    # Counter-proposal generator
│   ├── legal-missing/SKILL.md      # Missing-protections finder
│   ├── legal-nda/SKILL.md          # NDA generator (mutual/one-way/employee/vendor)
│   ├── legal-terms/SKILL.md        # Terms of service generator (GDPR/CCPA compliant)
│   ├── legal-privacy/SKILL.md      # Privacy policy generator
│   ├── legal-agreement/SKILL.md    # Business agreement generator (SOW, MSA, etc.)
│   ├── legal-freelancer/SKILL.md   # Freelancer-perspective contract review
│   ├── legal-compliance/SKILL.md   # Compliance gap analysis (GDPR, CCPA, ADA, PCI-DSS)
│   └── legal-report-pdf/SKILL.md   # PDF report with score gauges + risk charts
├── agents/
│   ├── legal-clauses.md            # Clause identification & categorization
│   ├── legal-risks.md              # Risk scoring agent
│   ├── legal-compliance.md         # Regulatory compliance agent
│   ├── legal-terms.md              # Terms & obligations mapper
│   └── legal-recommendations.md    # Fix recommendations agent
├── scripts/
│   └── generate_legal_pdf.py       # ReportLab PDF generation
├── templates/
│   └── contract-review-template.md # Output report template
├── install.sh                      # One-line installer
├── uninstall.sh                    # Clean uninstaller
└── generate_sample_contract.py     # Demo contract generator
```

## Commands Available After Install

| Command | Purpose |
|---------|---------|
| `/legal review <file>` | Full review: Contract Safety Score + clause analysis + recommendations |
| `/legal risks <file>` | Risk analysis with severity scoring and financial exposure |
| `/legal compare <file1> <file2>` | Version comparison flagging additions/removals |
| `/legal plain <file>` | Plain-English translation of every clause |
| `/legal negotiate <file>` | Counter-proposals with replacement language |
| `/legal missing <file>` | Missing-protections audit |
| `/legal nda <description>` | Generate custom NDA |
| `/legal terms <url>` | Generate terms of service |
| `/legal privacy <url>` | Generate privacy policy |
| `/legal agreement <type>` | Generate business agreement |
| `/legal freelancer <file>` | Freelancer-perspective review |
| `/legal compliance <url>` | Compliance gap analysis |
| `/legal report-pdf` | Generate professional PDF report |

## Installation / Uninstallation

```bash
# Install (curl)
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# Install (local)
./install.sh

# Uninstall
./uninstall.sh
```

## PDF Generation

The PDF skill (`/legal report-pdf`) requires Python 3.8+ and ReportLab:

```bash
pip3 install reportlab
python3 scripts/generate_legal_pdf.py
```

`generate_sample_contract.py` generates a demo contract for testing.

## How the Flagship `/legal review` Works

Five agents launch in parallel, each weighted by contribution:

| Agent | Role | Weight |
|-------|------|--------|
| legal-clauses | Identifies and categorizes every clause | 20% |
| legal-risks | Scores each clause for risk | 25% |
| legal-compliance | Flags regulatory issues | 20% |
| legal-terms | Maps obligations, deadlines, triggers | 15% |
| legal-recommendations | Generates specific fix language | 20% |

Output: Contract Safety Score (0–100), risk dashboard, clause-by-clause breakdown, missing protections, obligations timeline, compliance flags, negotiation priorities.

## Key Conventions

- **Skills are self-contained**: each `SKILL.md` is a complete, deployable skill — do not split logic between a skill and an agent
- **Agents are specialists**: agents in `agents/` are invoked by skills (especially `legal-review`) and focus on one dimension of analysis
- **legal/SKILL.md is the router**: it dispatches based on the sub-command name; always keep the command table in sync when adding skills
- **No AI in scripts**: `generate_legal_pdf.py` is deterministic ReportLab code — do not add LLM calls to it
- **Disclaimer is mandatory**: every output must include the disclaimer that this is not legal advice and a licensed attorney should review before signing

## Requirements

- Claude Code with an active Anthropic API key
- Python 3.8+ (for PDF generation only — `pip3 install reportlab`)

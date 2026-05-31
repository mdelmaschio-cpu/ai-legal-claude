# CLAUDE.md — ai-legal-claude

This file provides guidance to AI assistants working in this repository.

## Repository Purpose

An AI-powered legal assistant for Claude Code: **14 slash commands** across contract analysis, document generation, and compliance checking. The flagship `/legal review` uses 5 parallel agents to produce a Contract Safety Score (0-100). Part of the Claude Code Skills Series.

## Repository Structure

```
ai-legal-claude/
├── README.md                       # Full documentation and command reference
├── install.sh                      # One-line installer → ~/.claude/
├── uninstall.sh                    # Clean uninstaller
├── generate_sample_contract.py     # Generates test PDF contracts
├── sample-contract.pdf             # Example contract for testing skills
├── assets/
│   └── banner.svg                  # README banner
│
├── legal/
│   └── SKILL.md                    # Main command router — dispatches all /legal commands
│
├── skills/                         # 13 individual skill files
│   ├── legal-review/SKILL.md       # Flagship: 5-agent parallel review
│   ├── legal-risks/SKILL.md        # Deep risk analysis + financial exposure estimates
│   ├── legal-compare/SKILL.md      # Side-by-side version diff
│   ├── legal-plain/SKILL.md        # Legalese → plain English
│   ├── legal-negotiate/SKILL.md    # Counter-proposals with replacement language
│   ├── legal-missing/SKILL.md      # Missing protections finder
│   ├── legal-nda/SKILL.md          # Custom NDA generator (mutual, one-way, employee, vendor)
│   ├── legal-terms/SKILL.md        # Terms of service generator (GDPR/CCPA compliant)
│   ├── legal-privacy/SKILL.md      # Privacy policy generator
│   ├── legal-agreement/SKILL.md    # Business agreements (freelancer, partnership, SOW, MSA)
│   ├── legal-freelancer/SKILL.md   # Freelancer-perspective review
│   ├── legal-compliance/SKILL.md   # GDPR/CCPA/ADA/PCI-DSS/CAN-SPAM/SOC 2 gap analysis
│   └── legal-report-pdf/SKILL.md  # PDF report with score gauges + risk charts
│
├── agents/                         # 5 specialist agents for /legal review
│   ├── legal-clauses.md            # Clause Analyst (20% weight)
│   ├── legal-risks.md              # Risk Assessor (25% weight)
│   ├── legal-compliance.md         # Compliance Checker (20% weight)
│   ├── legal-terms.md              # Terms Mapper (15% weight)
│   └── legal-recommendations.md   # Recommendations Engine (20% weight)
│
├── scripts/
│   └── generate_legal_pdf.py       # ReportLab: produces score gauges + charts
│
└── templates/
    └── contract-review-template.md # Standardized report structure
```

## All 14 Commands

| Command | Purpose |
|---------|---------|
| `/legal review <file>` | Full 5-agent review → Contract Safety Score |
| `/legal risks <file>` | Deep risk analysis with financial exposure |
| `/legal compare <file1> <file2>` | Side-by-side version diff |
| `/legal plain <file>` | Translate every clause to plain English |
| `/legal negotiate <file>` | Generate counter-proposals with replacement language |
| `/legal missing <file>` | Find missing protections |
| `/legal nda <description>` | Generate custom NDA |
| `/legal terms <url>` | Generate terms of service |
| `/legal privacy <url>` | Generate privacy policy |
| `/legal agreement <type>` | Generate business agreements |
| `/legal freelancer <file>` | Freelancer-perspective review |
| `/legal compliance <url>` | GDPR/CCPA/ADA/PCI-DSS gap analysis |
| `/legal report-pdf` | PDF report with score gauges |

## The 5-Agent Review Architecture

`/legal review` launches 5 agents in parallel:

| Agent | File | Weight | Role |
|-------|------|--------|------|
| Clause Analyst | `legal-clauses.md` | 20% | Identifies and categorizes all clauses |
| Risk Assessor | `legal-risks.md` | 25% | Scores each clause for risk |
| Compliance Checker | `legal-compliance.md` | 20% | Flags regulatory issues |
| Terms Mapper | `legal-terms.md` | 15% | Maps obligations, deadlines, triggers |
| Recommendations Engine | `legal-recommendations.md` | 20% | Generates specific fixes |

All 5 results aggregate into a single Contract Safety Score (0-100).

## Installation

```bash
# One-line install
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# For PDF generation only
pip3 install reportlab

# Test with sample contract
python3 generate_sample_contract.py
/legal review sample-contract.pdf
```

## Requirements

- Claude Code with active Anthropic API key
- Python 3.8+ (PDF generation only)
- `reportlab` package (PDF generation only)

## Development Conventions

- All SKILL.md files use only `name` and `description` frontmatter fields — no extras
- The main router (`legal/SKILL.md`) handles command routing — individual skills in `skills/` are self-contained
- Output format is markdown — no XML tags
- The disclaimer ("for informational purposes only, not legal advice") is NON-NEGOTIABLE in all output
- Agent weight percentages (20/25/20/15/20) must sum to 100 — do not change without updating the scoring logic
- `generate_legal_pdf.py` uses only `reportlab` — do not add additional Python dependencies

## Testing

```bash
# Generate a sample contract
python3 generate_sample_contract.py

# Test the flagship command
# (in a Claude Code session with skills installed)
/legal review sample-contract.pdf
```

## Important Notes for AI Assistants

- Legal output requires the disclaimer in every response — enforce this across all 14 skills
- The Contract Safety Score is computed by the 5 agents — maintain their weight distribution
- CUAD's 41 risk categories are the ground truth for clause classification (not ad-hoc categories)
- Position-aware review is essential: customer vs vendor, buyer vs seller, receiving vs disclosing party — each changes what's flagged as risky
- Market benchmarks (e.g., 12-month liability cap = standard) are US-law defaults — flag international contracts
- `generate_sample_contract.py` creates a realistic test PDF — use it to verify end-to-end functionality
- Do NOT make claims that AI review replaces attorney review — always recommend professional counsel for material terms

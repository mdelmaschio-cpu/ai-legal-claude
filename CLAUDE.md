# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A suite of 14 Claude Code skills for AI-powered legal document analysis and generation. It enables users to review contracts, flag risks, generate NDAs and agreements, check compliance, and produce professional PDF reports — all from Claude Code without leaving the terminal.

This is **not a legal service**. Every output carries a mandatory disclaimer recommending consultation with a licensed attorney.

## Repository Structure

```
ai-legal-claude/
├── legal/
│   └── SKILL.md                    # Main dispatcher / orchestrator skill
├── skills/                         # 13 sub-skills, one per command
│   ├── legal-review/SKILL.md       # Flagship: full contract review via 5 parallel agents
│   ├── legal-risks/SKILL.md        # Clause-by-clause risk scoring
│   ├── legal-compare/SKILL.md      # Side-by-side diff of two contracts
│   ├── legal-plain/SKILL.md        # Legalese → plain English translation
│   ├── legal-negotiate/SKILL.md    # Counter-proposal generator
│   ├── legal-missing/SKILL.md      # Missing protections finder
│   ├── legal-nda/SKILL.md          # Custom NDA generator (mutual/one-way/employee/vendor)
│   ├── legal-terms/SKILL.md        # Terms of service generator
│   ├── legal-privacy/SKILL.md      # Privacy policy generator
│   ├── legal-agreement/SKILL.md    # Business agreements (SOW, MSA, partnership, etc.)
│   ├── legal-freelancer/SKILL.md   # Freelancer/contractor-perspective review
│   ├── legal-compliance/SKILL.md   # GDPR/CCPA/ADA/PCI-DSS/SOC 2 gap analysis
│   └── legal-report-pdf/SKILL.md   # Professional PDF report via ReportLab
├── agents/                         # 5 specialist agents used by /legal review
│   ├── legal-clauses.md            # Clause identification and categorisation
│   ├── legal-risks.md              # Risk scoring per clause
│   ├── legal-compliance.md         # Regulatory compliance checks
│   ├── legal-terms.md              # Obligations, deadlines, triggers mapping
│   └── legal-recommendations.md    # Specific fix / alternative language suggestions
├── scripts/
│   └── generate_legal_pdf.py       # PDF report generation (ReportLab)
├── templates/
│   └── contract-review-template.md # Output structure template
├── assets/
│   └── banner.svg
├── install.sh                      # One-line installer → copies skills to ~/.claude/skills/
├── uninstall.sh
└── README.md
```

## Installation

```bash
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash
```

The script copies all skill directories into `~/.claude/skills/` so Claude Code picks them up automatically.

### Requirements

- **Claude Code** with active Anthropic API key
- **Python 3.8+** — only needed for PDF generation
- **reportlab** — `pip3 install reportlab` — only needed for `/legal report-pdf`

No other dependencies.

## Command Reference

| Command | Routes to | What it does |
|---------|-----------|-------------|
| `/legal review <file>` | `legal-review` | Full review: 5 parallel agents → Contract Safety Score 0-100 + clause analysis |
| `/legal risks <file>` | `legal-risks` | Severity-scored risk analysis per clause |
| `/legal compare <file1> <file2>` | `legal-compare` | Side-by-side diff, flags dangerous changes |
| `/legal plain <file>` | `legal-plain` | Translates legalese to plain English |
| `/legal negotiate <file>` | `legal-negotiate` | Counter-proposals with replacement language |
| `/legal missing <file>` | `legal-missing` | Identifies absent protective clauses |
| `/legal nda <description>` | `legal-nda` | Generates a custom NDA |
| `/legal terms <url>` | `legal-terms` | Terms of service (GDPR/CCPA-compliant) |
| `/legal privacy <url>` | `legal-privacy` | Privacy policy from site scan |
| `/legal agreement <type>` | `legal-agreement` | Freelancer contract, partnership, SOW, MSA, etc. |
| `/legal freelancer <file>` | `legal-freelancer` | Contractor-specific risk review |
| `/legal compliance <url>` | `legal-compliance` | Gap analysis: GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2 |
| `/legal report-pdf` | `legal-report-pdf` | Generates a PDF with score gauges and risk charts |

## How `/legal review` Works

The flagship skill launches 5 AI agents in parallel:

| Agent | Role | Weight |
|-------|------|--------|
| Clause Analyst | Identifies and categorises every clause | 20% |
| Risk Assessor | Scores each clause for risk | 25% |
| Compliance Checker | Flags regulatory issues | 20% |
| Terms Mapper | Maps obligations, deadlines, triggers | 15% |
| Recommendations Engine | Generates specific fix language | 20% |

Results aggregate into a single Contract Safety Score (0–100) plus an eight-section report.

## Skill Architecture

### Dispatcher (`legal/SKILL.md`)

The main orchestrator. When the user types `/legal`, it:
1. Presents the full command menu
2. Parses the sub-command
3. Routes to the matching skill in `skills/`

### Sub-skills (`skills/*/SKILL.md`)

Each sub-skill is single-purpose. They accept one of three input forms:
- **File path** — read with the `Read` tool
- **Pasted text** — inline in the chat
- **URL** — fetched with `WebFetch`

### Output File Naming

Generated documents are saved in the working directory with date suffixes:
```
NDA-[party]-[YYYY-MM-DD].md
TERMS-OF-SERVICE-[company]-[YYYY-MM-DD].md
PRIVACY-POLICY-[company]-[YYYY-MM-DD].md
CONTRACT-REVIEW-[name]-[YYYY-MM-DD].md
```

## PDF Generation

`scripts/generate_legal_pdf.py` uses `reportlab` to render styled PDFs with:
- Score gauges (0-100 Contract Safety Score)
- Risk distribution charts (high/medium/low counts)
- Clause-by-clause analysis tables
- Prioritised action checklist

The `/legal report-pdf` skill calls this script automatically.

## Key Conventions

### Mandatory Disclaimer

Every skill output must begin with:

```
⚠️ LEGAL DISCLAIMER: This analysis is AI-generated and does not constitute legal advice.
It is intended as a starting point for review. Always consult a licensed attorney before
signing contracts or relying on generated legal documents.
```

### Risk Indicators

All risk assessments use three levels:
- 🔴 **High Risk** — requires immediate attention or legal escalation
- 🟡 **Medium Risk** — worth flagging, may be negotiable
- 🟢 **Low Risk** — standard language, no immediate action needed

### Tone Rules

- Professional but accessible — avoid jargon without explanation
- Always explain WHY something is risky, not just THAT it is
- Always propose specific alternative language alongside any red flag

## Modifying Skills

Each `SKILL.md` file is the complete instruction set for that command — routing, input handling, output format, and hard rules. To change how a skill behaves, edit its `SKILL.md`.

To add a new skill:
1. Create `skills/<skill-name>/SKILL.md`
2. Add routing entry to `legal/SKILL.md`'s routing table
3. Update the command menu in `legal/SKILL.md`
4. Update `README.md`

## Disclaimer

This project is for educational and informational purposes only. It does not provide legal advice and should not be used as a substitute for consultation with a licensed attorney.

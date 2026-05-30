# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

An AI-powered legal toolkit for Claude Code. Provides 14 commands via `/legal <command>` for contract review, document generation, and compliance analysis. The flagship `/legal review` launches 5 parallel specialized agents to produce a Contract Safety Score (0–100) with clause-by-clause analysis.

## Install / Uninstall

```bash
# One-line install (all 14 skills, 5 agents, PDF scripts)
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# Local install/uninstall
./install.sh
./uninstall.sh
```

**External dependency for PDF generation only:**
```bash
pip3 install reportlab
```
All other commands work without additional dependencies.

## Repository Architecture

```
ai-legal-claude/
├── legal/SKILL.md              # Entry point — routes all /legal <cmd> invocations
├── skills/                     # 13 sub-skill SKILL.md files
│   ├── legal-review/           # Flagship: launches 5 parallel agents
│   ├── legal-risks/            # Deep risk analysis with financial exposure estimates
│   ├── legal-compare/          # Side-by-side diff of two contract versions
│   ├── legal-plain/            # Plain-English translation of clauses
│   ├── legal-negotiate/        # Counter-proposal generator with replacement language
│   ├── legal-missing/          # Detects missing protective provisions
│   ├── legal-nda/              # NDA generator (mutual, one-way, employee, vendor)
│   ├── legal-terms/            # Terms of service generator
│   ├── legal-privacy/          # Privacy policy generator
│   ├── legal-agreement/        # Business agreement generator (SOW, MSA, partnerships)
│   ├── legal-compliance/       # GDPR/CCPA/ADA/PCI-DSS gap analysis
│   ├── legal-freelancer/       # Freelancer-perspective contract review
│   └── legal-report-pdf/       # PDF report with score gauges and risk charts
├── agents/                     # 5 sub-agents invoked in parallel by legal-review
│   ├── legal-clauses.md        # Clause identification and categorization (weight: 20%)
│   ├── legal-risks.md          # Risk scoring per clause (25%)
│   ├── legal-compliance.md     # Regulatory flag detection (20%)
│   ├── legal-terms.md          # Obligations and deadline mapping (15%)
│   └── legal-recommendations.md # Fix recommendations (20%)
├── scripts/
│   └── generate_legal_pdf.py  # ReportLab PDF generation
└── templates/
    └── contract-review-template.md
```

## Command Architecture

`legal/SKILL.md` is the router — it reads the command after `/legal` and delegates to the matching sub-skill in `skills/`. Only `legal-review` invokes the 5 agents in `agents/`; all other sub-skills operate independently.

## Key Commands

```
/legal review <file>         # Full review: safety score, clause analysis, redlines, next steps
/legal risks <file>          # Risk scoring with financial exposure per clause
/legal nda <description>     # Generate NDA
/legal compliance <url>      # Compliance gap analysis (GDPR, CCPA, ADA, PCI-DSS)
/legal report-pdf            # PDF report (requires reportlab)
```

## Output

All commands produce structured Markdown. `legal-review` aggregates the 5 agents' results into a single unified report. `legal-report-pdf` invokes `scripts/generate_legal_pdf.py` to produce a PDF with score gauges and risk charts from the review output.

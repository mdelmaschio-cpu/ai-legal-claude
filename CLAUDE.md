# CLAUDE.md — ai-legal-claude

This file provides guidance to Claude Code when working in this repository.

## Project Purpose

**AI Legal Assistant for Claude Code** is a Claude Code skills suite providing AI-powered contract review and legal document generation. It enables users to review contracts, flag risks, generate NDAs and other agreements, check regulatory compliance, negotiate terms, and produce client-ready PDF reports — all from within Claude Code.

The repo contains 14 skills organized under a single `/legal` orchestrator, 5 parallel subagents that execute during a full contract review, a PDF generation script, and an installer. The flagship command is `/legal review`, which launches 5 agents in parallel and returns a Contract Safety Score (0–100), clause-by-clause risk analysis, missing protections, and prioritized negotiation actions.

This is part of the "Claude Code Skills Series" alongside AI Marketing Suite and AI Sales Team.

## Repository Structure

```
ai-legal-claude/
├── README.md                           # Full command reference, architecture, use cases
├── install.sh                          # One-line installer (curl | bash)
├── uninstall.sh                        # Clean uninstaller
├── generate_sample_contract.py         # Sample contract generator for testing
├── sample-contract.pdf                 # Example contract for demo/testing
│
├── legal/
│   └── SKILL.md                        # Main orchestrator — command router for all /legal commands
│
├── skills/                             # 13 individual skill implementations
│   ├── legal-review/SKILL.md           # Full contract review with 5 parallel agents
│   ├── legal-risks/SKILL.md            # Deep risk analysis with financial exposure estimates
│   ├── legal-compare/SKILL.md          # Side-by-side comparison of two contract versions
│   ├── legal-plain/SKILL.md            # Plain English translation of legalese clauses
│   ├── legal-negotiate/SKILL.md        # Counter-proposal generator with replacement language
│   ├── legal-missing/SKILL.md          # Missing protections finder
│   ├── legal-nda/SKILL.md              # NDA generator (mutual, one-way, employee, vendor)
│   ├── legal-terms/SKILL.md            # Terms of service generator (GDPR/CCPA compliant)
│   ├── legal-privacy/SKILL.md          # Privacy policy generator (scans what site collects)
│   ├── legal-agreement/SKILL.md        # Business agreement generator (SOW, MSA, partnerships)
│   ├── legal-freelancer/SKILL.md       # Freelancer-perspective contract review
│   ├── legal-compliance/SKILL.md       # Compliance gap analysis (GDPR, CCPA, ADA, PCI-DSS)
│   └── legal-report-pdf/SKILL.md       # Professional PDF report with score gauges
│
├── agents/                             # 5 subagents launched in parallel by /legal review
│   ├── legal-clauses.md                # Clause Analyst (20% weight) — extract + categorize
│   ├── legal-risks.md                  # Risk Assessor (25% weight) — score each clause
│   ├── legal-compliance.md             # Compliance Checker (20% weight) — flag regulatory issues
│   ├── legal-terms.md                  # Terms Mapper (15% weight) — obligations + deadlines
│   └── legal-recommendations.md        # Recommendations Engine (20% weight) — specific fixes
│
├── scripts/
│   └── generate_legal_pdf.py           # PDF generation using ReportLab
│
├── templates/
│   └── contract-review-template.md     # Structured output template for reports
│
└── assets/
    └── banner.svg                      # Repository banner image
```

## All 14 Commands

### Contract Analysis
| Command | Purpose |
|---------|---------|
| `/legal review <file>` | Full review — 5 parallel agents, Contract Safety Score, clause-by-clause analysis |
| `/legal risks <file>` | Deep risk analysis with severity scoring and financial exposure estimates |
| `/legal compare <file1> <file2>` | Side-by-side diff of two contract versions, flags dangerous changes |
| `/legal plain <file>` | Translates every clause from legalese into plain English |
| `/legal negotiate <file>` | Generates specific counter-proposals with replacement language |
| `/legal missing <file>` | Finds protections that should be in the contract but aren't |

### Document Generation
| Command | Purpose |
|---------|---------|
| `/legal nda <description>` | Generates a custom NDA (mutual, one-way, employee, or vendor) |
| `/legal terms <url>` | Generates terms of service based on what the website does |
| `/legal privacy <url>` | Generates a privacy policy by scanning what data the site collects |
| `/legal agreement <type>` | Generates business agreements (freelancer, partnership, SOW, MSA) |
| `/legal freelancer <file>` | Specialized review from the freelancer's perspective |

### Compliance & Reporting
| Command | Purpose |
|---------|---------|
| `/legal compliance <url>` | Compliance gap analysis against GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2 |
| `/legal report-pdf` | Professional PDF report with score gauges and risk charts |

## How the `/legal review` Parallel Agent System Works

The `legal-review` skill launches 5 subagents in parallel:

| Agent | Weight | Role |
|-------|--------|------|
| Clause Analyst | 20% | Identifies and categorizes every clause using a 20-category taxonomy |
| Risk Assessor | 25% | Scores each clause 1–10 for risk severity |
| Compliance Checker | 20% | Flags regulatory issues (GDPR, CCPA, etc.) |
| Terms Mapper | 15% | Maps obligations, deadlines, and triggers |
| Recommendations Engine | 20% | Generates specific fix recommendations with replacement language |

Results aggregate into a unified Contract Safety Score (0–100). The Clause Analyst output is foundational — all other agents consume its clause inventory.

## Skill File Format

Each skill lives at `skills/<name>/SKILL.md`. Skills follow the standard Claude Code skill format:

```yaml
---
name: skill-name
description: Trigger-phrase-rich description with exact user phrases that should invoke this skill.
---

# Skill Title

[Instructions, workflows, output format]
```

The orchestrator at `legal/SKILL.md` acts as the command router — it reads the sub-command after `/legal` and routes to the appropriate skill.

## Agent File Format

Agent files in `agents/` describe parallel subagents invoked during review:
- Role and weight in the overall scoring system
- Step-by-step analysis process
- Clause taxonomy or risk categories to apply
- Structured output format with tables, metrics, and summaries
- Handoff instructions (what other agents consume this agent's output)
- Standard legal disclaimer

## PDF Generation

`scripts/generate_legal_pdf.py` uses the `reportlab` library. This is the only component with an external dependency.

**Requirements:**
- Python 3.8+
- `reportlab`: `pip3 install reportlab`

All other skills work without external dependencies.

## Installation

```bash
# Install all skills, agents, and scripts
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# Uninstall
./uninstall.sh
```

The installer copies skills to `~/.claude/skills/` and agents to `~/.claude/agents/`.

## Output Format Conventions

All review output uses **structured markdown only** — never XML tags. The standard output structure:

1. Document metadata (type, parties, risk level, status)
2. Pre-signing alerts (blank fields, missing exhibits)
3. Executive summary (1–2 paragraphs)
4. Key terms table (term, value, section reference)
5. Red flags quick scan table
6. Risk analysis sections (Critical / Important / Acceptable)
7. Missing provisions table with suggested language
8. Internal consistency issues
9. Negotiation priority table (ranked by importance and negotiability)
10. Legal disclaimer

Risk levels use plain labels: Critical, Important, Acceptable. Confidence indicators are used in findings — never present uncertain information as verified fact.

## Key Conventions for AI Assistants

### Legal Disclaimer — Non-Negotiable

Every review output, generated document, and compliance analysis must end with:

> "This review is for informational purposes only and does not constitute legal advice. Material terms should be reviewed by qualified legal counsel."

Never omit this disclaimer.

### Position-Awareness

Always determine which party the user represents before analyzing risk. Ask if unclear. The same clause is favorable or unfavorable depending on position (customer vs. vendor, buyer vs. seller, receiving party vs. disclosing party).

### No Fabrication

Only reference text that is actually in the document being reviewed. If a clause is absent, note it in the gap analysis — not in the risk analysis. Never invent clauses or cite sections that don't exist.

### Express Uncertainty

When interpretation is genuinely unclear, say so explicitly. "This clause is ambiguous — it could mean X or Y. A qualified attorney in [jurisdiction] would need to clarify."

### Jurisdiction Matters

Flag jurisdiction-dependent enforceability. Non-competes are generally void in California, North Dakota, Oklahoma, and Minnesota. Always note when governing law affects the analysis.

## Development Workflow

### Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`)
2. Follow the output format conventions (structured markdown, no XML)
3. Include the standard legal disclaimer in the output format
4. Add the command routing to `legal/SKILL.md`
5. Update README.md command table

### Testing

Use `generate_sample_contract.py` to generate test contracts:
```bash
python3 generate_sample_contract.py
```

Or use the provided `sample-contract.pdf` for end-to-end testing.

## Important Files

| File | Role |
|------|------|
| `legal/SKILL.md` | Main orchestrator and command router |
| `agents/legal-clauses.md` | Clause taxonomy and analysis process — foundational to the review pipeline |
| `agents/legal-risks.md` | Risk scoring rubric and financial exposure estimation |
| `scripts/generate_legal_pdf.py` | PDF generation (requires `reportlab`) |
| `install.sh` | Installer — copies skills and agents to `~/.claude/` |
| `templates/contract-review-template.md` | Standard output structure for reports |

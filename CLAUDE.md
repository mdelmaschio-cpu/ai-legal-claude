# CLAUDE.md — ai-legal-claude

This file provides guidance to AI assistants working in this repository.

## Project Overview

**ai-legal-claude** is a Claude Code skills package for AI-powered legal document analysis and generation. It ships 14 slash-command skills, 5 parallel sub-agents, a PDF generation script, and an installer — all usable directly from Claude Code.

This is **not** legal advice. It is an automation tool for contract review workflows.

## Repository Layout

```
ai-legal-claude/
├── legal/
│   └── SKILL.md              # Entry-point skill — routes /legal <subcommand>
├── skills/
│   ├── legal-review/SKILL.md # Full review: 5 agents, Contract Safety Score
│   ├── legal-risks/SKILL.md  # Deep risk analysis with financial exposure
│   ├── legal-compare/SKILL.md
│   ├── legal-plain/SKILL.md  # Plain-English translation
│   ├── legal-negotiate/SKILL.md
│   ├── legal-missing/SKILL.md
│   ├── legal-nda/SKILL.md
│   ├── legal-terms/SKILL.md
│   ├── legal-privacy/SKILL.md
│   ├── legal-agreement/SKILL.md
│   ├── legal-freelancer/SKILL.md
│   ├── legal-compliance/SKILL.md
│   └── legal-report-pdf/SKILL.md
├── agents/
│   ├── legal-clauses.md        # Clause identification agent (20% weight)
│   ├── legal-risks.md          # Risk scoring agent (25% weight)
│   ├── legal-compliance.md     # Regulatory compliance agent (20% weight)
│   ├── legal-terms.md          # Obligations & deadlines agent (15% weight)
│   └── legal-recommendations.md # Fix recommendations agent (20% weight)
├── scripts/
│   └── generate_legal_pdf.py  # PDF report generator (ReportLab)
├── templates/
│   └── contract-review-template.md
├── assets/
│   └── banner.svg
├── install.sh
├── uninstall.sh
└── README.md
```

## Skill Architecture

### Command Router

`/legal` (in `legal/SKILL.md`) routes user invocations to the correct sub-skill:

| Command | Skill |
|---------|-------|
| `/legal review <file>` | `skills/legal-review/SKILL.md` |
| `/legal risks <file>` | `skills/legal-risks/SKILL.md` |
| `/legal compare <file1> <file2>` | `skills/legal-compare/SKILL.md` |
| `/legal plain <file>` | `skills/legal-plain/SKILL.md` |
| `/legal negotiate <file>` | `skills/legal-negotiate/SKILL.md` |
| `/legal missing <file>` | `skills/legal-missing/SKILL.md` |
| `/legal nda <description>` | `skills/legal-nda/SKILL.md` |
| `/legal terms <url>` | `skills/legal-terms/SKILL.md` |
| `/legal privacy <url>` | `skills/legal-privacy/SKILL.md` |
| `/legal agreement <type>` | `skills/legal-agreement/SKILL.md` |
| `/legal freelancer <file>` | `skills/legal-freelancer/SKILL.md` |
| `/legal compliance <url>` | `skills/legal-compliance/SKILL.md` |
| `/legal report-pdf` | `skills/legal-report-pdf/SKILL.md` |

### Parallel Agent Pattern (`/legal review`)

`legal-review` launches 5 agents simultaneously with weighted contributions:

| Agent | File | Weight |
|-------|------|--------|
| Clause Analyst | `agents/legal-clauses.md` | 20% |
| Risk Assessor | `agents/legal-risks.md` | 25% |
| Compliance Checker | `agents/legal-compliance.md` | 20% |
| Terms Mapper | `agents/legal-terms.md` | 15% |
| Recommendations Engine | `agents/legal-recommendations.md` | 20% |

Results are aggregated into a single report with a 0–100 Contract Safety Score.

## File Conventions

- All skill files use YAML frontmatter (`name`, `description`) followed by Markdown instructions
- Agent files are Markdown documents defining role, focus, and output format
- No binary artifacts are committed (PDFs are generated at runtime)
- The installer copies skill files into `~/.claude/skills/` and agents into `~/.claude/agents/`

## Development Workflow

There is **no build step**. All content is Markdown files.

### Adding or Modifying a Skill

1. Edit the target `skills/<skill-name>/SKILL.md`
2. Update `legal/SKILL.md` if the routing table changes
3. Test by invoking the command in a Claude Code session with a sample contract
4. Keep `description` in frontmatter trigger-phrase-rich for correct skill selection

### Adding a New Agent

1. Create `agents/<role>.md` following the pattern of existing agents
2. Update `skills/legal-review/SKILL.md` to reference the new agent and adjust weights
3. Document the new agent's role in the parallel dispatch table above

### PDF Generation

The PDF script uses **ReportLab** (Python 3.8+). Run it standalone:

```bash
pip3 install reportlab
python3 scripts/generate_legal_pdf.py
```

The script reads structured review output and produces a PDF with score gauges and risk charts. It is invoked by `/legal report-pdf` after a review.

## Installation

```bash
# Install all 14 skills and 5 agents
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# Or run locally
./install.sh

# Uninstall
./uninstall.sh
```

## Key Conventions

- **Disclaimer always**: Every output must include the legal disclaimer (this tool is not a substitute for a licensed attorney)
- **File references**: Skills expect contract files as local paths or URLs; always validate the file exists before processing
- **Output format**: Reviews return structured Markdown with a score, risk counts, and clause-by-clause breakdown — never raw freeform text
- **PDF only on request**: The PDF report is a separate command; do not auto-generate it during review

## Dependencies

| Dep | Purpose | Required |
|-----|---------|----------|
| Claude Code | Runtime | Yes |
| Python 3.8+ | PDF generation only | Optional |
| reportlab | PDF generation only | Optional (`pip3 install reportlab`) |

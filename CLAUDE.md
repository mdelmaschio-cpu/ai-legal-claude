# CLAUDE.md — AI Legal Assistant for Claude Code

This file provides guidance for AI assistants (Claude and others) working in this repository.

## Overview

`ai-legal-claude` is an AI-powered legal assistance toolkit that runs entirely within Claude Code. It provides contract review, risk analysis, legal document generation, compliance checking, and PDF report generation through a suite of 14 slash-command skills backed by 5 specialized agents.

The project is installed into `~/.claude/skills/` and `~/.claude/agents/` and is invoked via `/legal <subcommand>` within Claude Code sessions. It is **not** a standalone application — it extends the Claude Code environment.

> **Disclaimer**: This tool is for educational and informational purposes only. It does not constitute legal advice and is not a substitute for consultation with a licensed attorney.

---

## Repository Structure

```
ai-legal-claude/
├── legal/
│   └── SKILL.md                    # Main orchestrator — routes /legal commands
├── skills/                         # 13 individual skill definitions
│   ├── legal-review/SKILL.md       # Full contract review (5 parallel agents)
│   ├── legal-risks/SKILL.md        # Deep risk analysis with financial exposure
│   ├── legal-compare/SKILL.md      # Side-by-side contract version comparison
│   ├── legal-plain/SKILL.md        # Plain-English translation of legalese
│   ├── legal-negotiate/SKILL.md    # Counter-proposal generator
│   ├── legal-missing/SKILL.md      # Missing protections finder
│   ├── legal-nda/SKILL.md          # NDA generator (mutual, one-way, employee, vendor)
│   ├── legal-terms/SKILL.md        # Terms of service generator (GDPR/CCPA)
│   ├── legal-privacy/SKILL.md      # Privacy policy generator
│   ├── legal-agreement/SKILL.md    # Business agreement generator
│   ├── legal-compliance/SKILL.md   # Compliance gap analysis
│   ├── legal-freelancer/SKILL.md   # Freelancer-perspective contract review
│   └── legal-report-pdf/SKILL.md   # PDF report generator
├── agents/                         # 5 specialized sub-agents
│   ├── legal-clauses.md            # Clause identification and categorization
│   ├── legal-risks.md              # Risk scoring per clause
│   ├── legal-compliance.md         # Regulatory compliance checking
│   ├── legal-terms.md              # Obligations, deadlines, and triggers mapping
│   └── legal-recommendations.md    # Generates specific fix recommendations
├── scripts/
│   └── generate_legal_pdf.py       # PDF generation using ReportLab
├── templates/
│   └── contract-review-template.md # Report template used by PDF skill
├── assets/
│   └── banner.svg                  # README banner image
├── generate_sample_contract.py     # Helper script to generate test contracts
├── sample-contract.pdf             # Sample contract for testing
├── install.sh                      # One-command installer (curl-safe)
├── uninstall.sh                    # Clean uninstaller
└── README.md
```

---

## Key Files

### `legal/SKILL.md`
The main orchestrator. When a user runs `/legal <subcommand>`, Claude Code loads this skill, which routes the command to the appropriate sub-skill. This is the entry point for all `/legal` commands.

### `skills/<name>/SKILL.md`
Each sub-skill is defined in its own `SKILL.md` file inside a named directory under `skills/`. These files contain the prompt instructions that Claude follows when that skill is invoked. Every skill file follows the Claude Code skill format.

### `agents/*.md`
The five agent files define specialized sub-agents used by `/legal review`. During a full review, all five agents run in parallel:
- **legal-clauses.md** — Identifies and categorizes every clause (20% weight)
- **legal-risks.md** — Scores each clause for risk severity (25% weight)
- **legal-compliance.md** — Flags regulatory issues — GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2 (20% weight)
- **legal-terms.md** — Maps obligations, deadlines, and triggers (15% weight)
- **legal-recommendations.md** — Generates specific fix recommendations (20% weight)

Agent outputs are aggregated into a single unified report with a Contract Safety Score (0–100).

### `scripts/generate_legal_pdf.py`
Python script using `reportlab` to produce client-ready PDF reports. Called by the `legal-report-pdf` skill. Requires Python 3.8+ and `reportlab`.

### `templates/contract-review-template.md`
Markdown template that defines the structure of the contract review report, including score gauges, risk charts, and prioritized actions.

### `install.sh`
Bash installer that is safe to pipe through `curl | bash`. It:
1. Detects whether it is running from a local clone or from a remote curl pipe
2. Clones the repository to a temp directory if running remotely (requires `git`)
3. Copies all skill files to `~/.claude/skills/`
4. Copies all agent files to `~/.claude/agents/`
5. Copies Python scripts and templates into `~/.claude/skills/legal/`
6. Checks for Python 3 and `reportlab`
7. Cleans up any temp files

---

## Installation and Setup

### One-Command Install (Recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash
```

### Local Install

```bash
git clone https://github.com/mdelmaschio-cpu/ai-legal-claude
cd ai-legal-claude
bash install.sh
```

### Prerequisites
- Claude Code with an active Anthropic API key
- `git` (required for remote installation)
- Python 3.8+ (only needed for PDF reports)
- `reportlab` Python package: `pip3 install reportlab` (only needed for PDF reports)

### What Gets Installed

| Destination | Contents |
|-------------|----------|
| `~/.claude/skills/legal/` | Main orchestrator SKILL.md, scripts/, templates/ |
| `~/.claude/skills/legal-*/` | 13 individual sub-skill SKILL.md files |
| `~/.claude/agents/` | 5 agent markdown files |

### Uninstall

```bash
bash uninstall.sh
# or remotely:
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/uninstall.sh | bash
```

---

## Available Commands

All commands are invoked within a Claude Code session using the `/legal` prefix.

### Contract Analysis

| Command | Description |
|---------|-------------|
| `/legal review <file>` | **Flagship** — Full review with 5 parallel agents. Returns Contract Safety Score (0–100), clause-by-clause analysis, missing protections, obligations timeline, compliance flags, and negotiation priorities. |
| `/legal risks <file>` | Deep risk analysis with severity scoring per clause; estimates financial exposure. |
| `/legal compare <file1> <file2>` | Side-by-side comparison of two contract versions; flags additions, removals, and dangerous changes. |
| `/legal plain <file>` | Translates every clause from legalese into plain English. |
| `/legal negotiate <file>` | Generates specific counter-proposals with replacement language for unfavorable clauses. |
| `/legal missing <file>` | Identifies protections that should be present but are absent from the contract. |
| `/legal freelancer <file>` | Specialized review from the freelancer/contractor perspective; flags common contractor traps. |

### Document Generation

| Command | Description |
|---------|-------------|
| `/legal nda <description>` | Generates a custom NDA (mutual, one-way, employee, or vendor). |
| `/legal terms <url>` | Generates terms of service based on what the website does; GDPR/CCPA compliant. |
| `/legal privacy <url>` | Generates a privacy policy by analyzing what data the site collects. |
| `/legal agreement <type>` | Generates business agreements — freelancer contracts, partnerships, SOWs, MSAs, and more. |

### Compliance and Reporting

| Command | Description |
|---------|-------------|
| `/legal compliance <url>` | Compliance gap analysis for GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2. |
| `/legal report-pdf` | Generates a professional PDF report with score gauges, risk charts, and prioritized actions. |

---

## How `/legal review` Works

The flagship command launches 5 AI agents in parallel:

| Agent | Role | Weight |
|-------|------|--------|
| Clause Analyst (`legal-clauses.md`) | Identifies and categorizes every clause | 20% |
| Risk Assessor (`legal-risks.md`) | Scores each clause for risk | 25% |
| Compliance Checker (`legal-compliance.md`) | Flags regulatory issues | 20% |
| Terms Mapper (`legal-terms.md`) | Maps obligations, deadlines, and triggers | 15% |
| Recommendations Engine (`legal-recommendations.md`) | Generates specific fixes | 20% |

Outputs are aggregated into a unified report containing:
1. Contract Safety Score (0–100) with letter grade
2. Risk Dashboard (high/medium/low clause counts)
3. Clause-by-clause analysis with plain-English explanations and fix recommendations
4. Missing protections
5. Obligations timeline
6. Compliance flags
7. Negotiation priorities (ranked)
8. Next steps checklist

---

## Skill Architecture

Skills in this repository follow the Claude Code skill format:
- Each skill lives in its own directory under `skills/` with a single `SKILL.md` file
- The main orchestrator at `legal/SKILL.md` acts as a router
- Sub-skills are self-contained prompt documents; they may reference agents but do not import each other
- Agents under `agents/` are standalone documents that can be invoked independently or as part of the orchestrated review flow

### Adding a New Skill

1. Create `skills/legal-<name>/SKILL.md` following the pattern of existing skill files
2. Register the skill name in `install.sh` in the `SKILLS` array
3. Update `legal/SKILL.md` to route the new `/legal <name>` command
4. Document the command in `README.md`

### Adding a New Agent

1. Create `agents/legal-<name>.md` following the pattern of existing agent files
2. Register the agent name in `install.sh` in the `AGENTS` array
3. Reference the agent in the relevant skill's `SKILL.md`

---

## Development Workflow

### Testing Skills Locally

1. Clone the repository
2. Run `bash install.sh` to install the current working copy into `~/.claude/`
3. Open Claude Code and test commands: `/legal review sample-contract.pdf`
4. Edit `skills/<name>/SKILL.md` or `agents/<name>.md` and re-run `install.sh` to pick up changes

### Testing PDF Generation

```bash
pip3 install reportlab
python3 scripts/generate_legal_pdf.py
```

### Generating a Sample Contract for Testing

```bash
python3 generate_sample_contract.py
```

This produces a `sample-contract.pdf` suitable for running through `/legal review`.

---

## Conventions

- **Skill files** are always named `SKILL.md` (uppercase), placed inside a directory named after the skill
- **Agent files** are named in lowercase with hyphens: `legal-<role>.md`
- **Commands** follow the pattern `/legal <verb> [argument]`
- **All skill and agent content** is plain Markdown; no code execution occurs inside skill files themselves
- **Python scripts** live in `scripts/` and are stand-alone; they do not import from each other
- **Templates** live in `templates/` and are Markdown files used as structural scaffolding by scripts
- The installer (`install.sh`) is the source of truth for what gets deployed; any new file added to the repo must also be added to the installer to be picked up
- Maintain backward compatibility: existing command names (`/legal review`, etc.) must not change without updating documentation

---

## Compliance Frameworks Covered

The `/legal compliance` skill checks against:
- **GDPR** — EU General Data Protection Regulation
- **CCPA** — California Consumer Privacy Act
- **ADA** — Americans with Disabilities Act
- **PCI-DSS** — Payment Card Industry Data Security Standard
- **CAN-SPAM** — Email marketing compliance
- **SOC 2** — Service Organization Control 2

---

## Related Projects

- [AI Marketing Suite](https://github.com/zubair-trabzada/ai-marketing-claude) — companion Claude Code skill suite
- [AI Sales Team](https://github.com/zubair-trabzada/ai-sales-team-claude) — companion Claude Code skill suite
- [legal-redline-tools](https://github.com/evolsb/legal-redline-tools) — generates tracked-changes Word docs and redline PDFs from structured JSON output

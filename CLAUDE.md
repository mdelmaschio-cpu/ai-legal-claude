# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

**ai-legal-claude** is a Claude Code skills package that installs 14 AI-powered legal slash commands into Claude Code. It provides contract review, risk analysis, legal document generation (NDAs, Terms of Service, Privacy Policies, business agreements), compliance auditing, and PDF report generation — all invokable via `/legal <subcommand>` inside Claude Code sessions.

The flagship `/legal review` command spawns 5 parallel subagents that perform independent analysis passes (clause identification, risk scoring, compliance checking, terms mapping, and recommendations), then aggregates their output into a scored contract review report.

This is a **documentation-only plugin** — no application to run, no build system, no test suite. All behavior is defined in Markdown `SKILL.md` and agent `.md` files. The only executable code is a single Python script for PDF generation.

This repo is part of a series by the same author: `ai-marketing-claude`, `ai-sales-team-claude`, `ai-legal-claude`.

## Repository Structure

```
ai-legal-claude/
├── legal/
│   └── SKILL.md                     # Main orchestrator — routes all 14 /legal sub-commands
├── skills/                          # 13 sub-skills, one per /legal sub-command
│   ├── legal-review/SKILL.md        # Flagship: full contract review via 5 parallel subagents
│   ├── legal-risks/SKILL.md         # Deep clause-by-clause risk scoring (10 categories)
│   ├── legal-compare/SKILL.md       # Side-by-side diff of two contract versions
│   ├── legal-plain/SKILL.md         # Legalese-to-plain-English translation
│   ├── legal-negotiate/SKILL.md     # Counter-proposal generator with replacement language
│   ├── legal-missing/SKILL.md       # Identifies missing protections
│   ├── legal-nda/SKILL.md           # Custom NDA generator (mutual, one-way, employee, vendor)
│   ├── legal-terms/SKILL.md         # Terms of Service generator (GDPR/CCPA compliant)
│   ├── legal-privacy/SKILL.md       # Privacy policy generator
│   ├── legal-agreement/SKILL.md     # Business agreement generator (freelancer, partnership, SOW, MSA)
│   ├── legal-compliance/SKILL.md    # Compliance gap analysis (GDPR, CCPA, ADA, PCI-DSS, etc.)
│   ├── legal-freelancer/SKILL.md    # Specialized freelancer/contractor contract review
│   └── legal-report-pdf/SKILL.md    # Orchestrates PDF report generation via Python script
├── agents/                          # 5 subagents launched in parallel by /legal review
│   ├── legal-clauses.md             # Clause Analysis agent (20% weight)
│   ├── legal-risks.md               # Risk Assessment agent (25% weight)
│   ├── legal-compliance.md          # Compliance Check agent (20% weight)
│   ├── legal-terms.md               # Terms & Obligations agent (15% weight)
│   └── legal-recommendations.md     # Recommendations agent (20% weight)
├── scripts/
│   └── generate_legal_pdf.py        # ReportLab PDF builder — accepts JSON, outputs PDF
├── templates/
│   └── contract-review-template.md  # Markdown report template with placeholder structure
├── assets/
│   └── banner.svg                   # GitHub README banner image
├── generate_sample_contract.py      # Dev utility: generates a flawed sample PDF for testing
├── sample-contract.pdf              # Pre-built sample contract for testing skills
├── install.sh                       # Installer (copies to ~/.claude/skills + ~/.claude/agents)
├── uninstall.sh                     # Removes all installed skills and agents
└── README.md                        # User-facing documentation with all 14 commands
```

## Installation and Setup

### Install from GitHub (end-user)

```bash
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash
```

### Install from local clone (development)

```bash
./install.sh
```

The installer copies:
- `legal/` and `skills/*/` → `~/.claude/skills/`
- `agents/*.md` → `~/.claude/agents/`
- `scripts/generate_legal_pdf.py` → `~/.claude/skills/legal/scripts/`
- `templates/` → `~/.claude/skills/legal/templates/`

After modifying any skill, agent, script, or template, re-run `./install.sh` to push changes to Claude Code.

### Uninstall

```bash
./uninstall.sh
```

### Python dependency (PDF reports only)

The PDF generation feature requires ReportLab. No other sub-command needs it.

```bash
pip3 install reportlab
```

This is the **only non-stdlib dependency** in the entire repository.

## Commands

### Slash Commands (invoked inside Claude Code after installation)

| Command | Description |
|---|---|
| `/legal review <file>` | Full contract review — launches 5 parallel subagents |
| `/legal risks <file>` | Deep clause-by-clause risk analysis |
| `/legal compare <file1> <file2>` | Side-by-side comparison of two contract versions |
| `/legal plain <file>` | Translate legalese to plain English |
| `/legal negotiate <file>` | Generate counter-proposals with replacement language |
| `/legal missing <file>` | Identify missing contractual protections |
| `/legal nda <description>` | Generate a custom NDA (mutual, one-way, employee, or vendor) |
| `/legal terms <url>` | Generate Terms of Service (GDPR/CCPA compliant) |
| `/legal privacy <url>` | Generate a Privacy Policy |
| `/legal agreement <type>` | Generate a business agreement (freelancer, partnership, SOW, MSA) |
| `/legal freelancer <file>` | Specialized freelancer/contractor contract review |
| `/legal compliance <url>` | Compliance gap analysis (GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2) |
| `/legal report-pdf` | Generate a PDF from the most recent review output |

### PDF generation script (called by `/legal report-pdf`)

```bash
python3 scripts/generate_legal_pdf.py <json_data_file> [output_path]
```

`output_path` defaults to `CONTRACT-REVIEW-REPORT.pdf` in the current directory.

The installed location is `~/.claude/skills/legal/scripts/generate_legal_pdf.py`. The `/legal report-pdf` skill searches for the script at several relative paths from the working directory.

### Sample contract generator (dev utility only)

```bash
python3 generate_sample_contract.py
```

**Gotcha**: This script has a hardcoded macOS absolute path at line 13 (`OUTPUT_PATH`). Update it before running locally. It is not part of the installed package and should not be distributed or run in production.

## Architecture Overview

### Skill invocation flow

```
User types /legal <subcommand> in Claude Code
        ↓
legal/SKILL.md (orchestrator) receives input
        ↓
Routes to skills/<legal-subcommand>/SKILL.md
        ↓
Sub-skill executes (reads files, fetches URLs, spawns agents)
        ↓
Output written to a timestamped .md file + displayed in session
```

### /legal review — parallel subagent architecture

The flagship command uses Claude's Agent tool to spawn 5 independent subagents concurrently:

| Agent file | Role | Weight |
|---|---|---|
| `agents/legal-clauses.md` | Clause Analysis — identifies and categorizes every clause | 20% |
| `agents/legal-risks.md` | Risk Assessment — scores clauses on 10 dimensions | 25% |
| `agents/legal-compliance.md` | Compliance Check — flags regulatory issues | 20% |
| `agents/legal-terms.md` | Terms & Obligations — maps duties, deadlines, triggers | 15% |
| `agents/legal-recommendations.md` | Recommendations — generates specific fix language | 20% |

The orchestrator (`skills/legal-review/SKILL.md`) runs four phases:
1. Contract ingestion via Claude's Read tool (or WebFetch for URLs)
2. Launch all 5 agents in parallel via the Agent tool
3. Aggregate results into a scored report
4. Present the report to the user and write it to a timestamped file

Weights must sum to 100%. If you edit scoring logic, keep the agent file references in `legal-review/SKILL.md` consistent with the actual filenames in `agents/`.

### Risk scoring methodology (legal-risks agent)

The `agents/legal-risks.md` file defines a 4-factor composite score:

| Factor | Weight |
|---|---|
| Severity | 40% |
| Likelihood | 25% |
| Financial Exposure | 20% |
| Asymmetry | 15% |

Round up when financial exposure is uncapped. Scores map to a 0-100 Contract Safety Score with letter grades: A+ (90-100), A (80-89), B (70-79), C (60-69), D (40-59), F (0-39).

### PDF report input schema

`generate_legal_pdf.py` accepts a JSON file with these keys:

```json
{
  "score": 72,
  "grade": "B",
  "grade_label": "Acceptable — Review Recommended",
  "details": {
    "type": "...", "parties": "...", "effective_date": "...",
    "term": "...", "total_value": "...", "governing_law": "..."
  },
  "executive_summary": "...",
  "risks": {
    "high": 3, "medium": 5, "low": 8,
    "clauses": [...]
  },
  "clauses": [
    {
      "risk": "high",
      "name": "...", "section": "...", "summary": "...",
      "risk_explanation": "...", "recommendation": "..."
    }
  ],
  "negotiation_priorities": [...],
  "missing_protections": [...],
  "next_steps": [...]
}
```

The `build_pdf()` function in `generate_legal_pdf.py` is the main entry point; it receives this dict and returns a file path. The PDF includes a score gauge, risk bar chart, clause analysis table, negotiation priorities, and a footer disclaimer.

## Key Files

### `legal/SKILL.md`

The main command router. Defines:
- Routing table for all 14 `/legal` sub-commands
- Input handling rules (file paths vs. URLs vs. descriptions)
- Output file naming conventions
- Disclaimer enforcement (every output must open with the legal disclaimer)
- Tone and style guidance

### `skills/legal-review/SKILL.md`

The most complex sub-skill. Defines the four-phase orchestration for the full contract review. Maintains consistency with the 5 agent files in `agents/`.

### `agents/legal-risks.md`

The most complex agent file. Defines the 10 risk categories, the 4-factor composite scoring methodology, poison pill detection heuristics, and detailed output format.

### `scripts/generate_legal_pdf.py`

The only executable code in the repository. Pure Python 3.8+, requires ReportLab. Accepts a JSON data file as its first argument; an optional second argument specifies the output path. Not required for any command except `/legal report-pdf`.

### `templates/contract-review-template.md`

The structural reference for every contract review output. All sections have placeholder comments explaining expected content. Used as the canonical format guide when editing review output structure.

### `install.sh` / `uninstall.sh`

Install/uninstall scripts. Work both locally and via curl-pipe-bash. The installer handles four destination directories under `~/.claude/`.

## Conventions

### Output file naming

All generated documents use uppercase type prefixes with the date:

```
NDA-[party-name]-[YYYY-MM-DD].md
TERMS-OF-SERVICE-[company]-[YYYY-MM-DD].md
PRIVACY-POLICY-[company]-[YYYY-MM-DD].md
CONTRACT-REVIEW-[name]-[YYYY-MM-DD].md
CONTRACT-COMPARISON-[YYYY-MM-DD].md
CONTRACT-REVIEW-REPORT.pdf
```

### Risk indicators

Every output uses consistent emoji throughout:
- 🔴 High Risk
- 🟡 Medium Risk
- 🟢 Low Risk

### Mandatory disclaimer

**Every output — analysis or generated document — must open with this exact text:**

```
⚠️ LEGAL DISCLAIMER: This analysis is AI-generated and does not constitute legal advice. Always consult a licensed attorney before signing contracts or relying on generated legal documents.
```

This is enforced in both the orchestrator (`legal/SKILL.md`) and each individual sub-skill. Do not remove or weaken it.

### SKILL.md frontmatter

Agent files under `agents/` do not use YAML frontmatter. Among sub-skills, only `skills/legal-nda/SKILL.md` has YAML frontmatter (`name`, `description`, `command` fields). This inconsistency is intentional — do not add frontmatter to agent files.

## How AI Assistants Should Work in This Repo

**This is a documentation-only plugin.** There is no application to run, no linter, no test suite. Manual validation via `sample-contract.pdf` and `/legal review` is the only quality gate.

### Modifying skill behavior

Edit the relevant `SKILL.md` or agent `.md` file. Changes take effect the next time the skill is invoked in Claude Code **after re-running `./install.sh`**. The install step is required — Claude Code reads from `~/.claude/skills/`, not from the repo working directory.

Scope of changes by file:
- Trigger phrases, routing logic, output format → `legal/SKILL.md` (orchestrator)
- Sub-command workflow steps → `skills/<legal-subcommand>/SKILL.md`
- Agent analysis methodology, scoring → `agents/*.md`
- Report template structure → `templates/contract-review-template.md`

### Modifying PDF output

Edit `scripts/generate_legal_pdf.py`. The `build_pdf()` function is the main entry point. ReportLab is the only non-stdlib dependency. After editing, reinstall with `./install.sh` so the updated script lands at `~/.claude/skills/legal/scripts/generate_legal_pdf.py`.

### Maintaining /legal review consistency

When editing `skills/legal-review/SKILL.md`, verify:
1. The 5 agent file references match the actual filenames in `agents/`
2. The scoring weights still sum to 100%
3. The output structure matches `templates/contract-review-template.md`

### Adding a new sub-command

1. Create `skills/legal-<name>/SKILL.md` with the sub-command's workflow
2. Add a routing entry in `legal/SKILL.md`
3. Add the output naming convention to the conventions section of this file
4. Run `./install.sh` to deploy

### Claude tools used at runtime

The skills rely on these Claude Code built-in tools — do not remove references to them from SKILL.md files:
- **Read** — ingests contract files from the filesystem
- **WebFetch** — fetches contracts from URLs; used by compliance and TOS/privacy generators for website scanning
- **Agent** — spawns the 5 parallel subagents in `/legal review`
- **Glob** — used by `/legal report-pdf` to find the most recent review output file

### Validation after edits

```bash
# 1. Reinstall after any file change
./install.sh

# 2. Test the full pipeline with the included sample contract
# (inside Claude Code)
/legal review ~/path/to/sample-contract.pdf

# 3. For PDF output
/legal report-pdf
```

There is no automated test suite. If end-to-end behavior is incorrect after edits, check:
- That `./install.sh` was re-run after the change
- That agent weight assignments in `legal-review/SKILL.md` still sum to 100%
- That the disclaimer is present in the modified skill's output format
- That `generate_legal_pdf.py` receives a JSON file matching the expected schema (for PDF issues)

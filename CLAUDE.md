# CLAUDE.md — AI Legal Assistant for Claude Code

## Repository Overview

This repository is a Claude Code skills pack that provides AI-powered legal document analysis and generation. It installs 14 slash commands (skills), 5 subagents, Python PDF generation scripts, and a report template into a user's Claude Code environment.

**Primary use cases:**
- Reviewing and risk-scoring contracts before signing
- Generating legal documents (NDAs, terms of service, privacy policies, freelancer agreements)
- Running website compliance gap audits (GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM)
- Producing professional PDF reports from analysis output

**Important:** This tool provides legal analysis as a starting point — it explicitly does NOT provide legal advice and always recommends attorney review before signing.

---

## Directory Structure

```
ai-legal-claude/
├── legal/
│   └── SKILL.md                    # Main command router and orchestrator
├── skills/
│   ├── legal-review/SKILL.md       # Flagship: full review with 5 parallel agents
│   ├── legal-risks/SKILL.md        # Deep clause-by-clause risk scoring
│   ├── legal-compare/SKILL.md      # Side-by-side comparison of two contracts
│   ├── legal-plain/SKILL.md        # Legalese-to-plain-English translation
│   ├── legal-negotiate/SKILL.md    # Counter-proposal generator with negotiation scripts
│   ├── legal-missing/SKILL.md      # Missing protections finder
│   ├── legal-nda/SKILL.md          # Custom NDA generator with annotations
│   ├── legal-terms/SKILL.md        # Terms of service generator
│   ├── legal-privacy/SKILL.md      # Privacy policy generator (GDPR/CCPA)
│   ├── legal-agreement/SKILL.md    # Business agreement generator
│   ├── legal-compliance/SKILL.md   # Website compliance audit (7 frameworks)
│   ├── legal-freelancer/SKILL.md   # Contractor-perspective contract review
│   └── legal-report-pdf/SKILL.md   # PDF report generator
├── agents/
│   ├── legal-clauses.md            # Subagent: clause extraction and categorization (20% weight)
│   ├── legal-risks.md              # Subagent: risk scoring 1-10 (25% weight)
│   ├── legal-compliance.md         # Subagent: regulatory compliance checks (20% weight)
│   ├── legal-terms.md              # Subagent: obligations/deadlines mapping (15% weight)
│   └── legal-recommendations.md    # Subagent: counter-proposals and negotiation (20% weight)
├── scripts/
│   └── generate_legal_pdf.py       # ReportLab PDF builder (CLI tool)
├── templates/
│   └── contract-review-template.md # Output template for /legal review
├── assets/
│   └── banner.svg                  # GitHub README banner
├── generate_sample_contract.py     # Dev utility: generate test contract PDFs
├── sample-contract.pdf             # Sample contract for testing
├── install.sh                      # One-command installer
├── uninstall.sh                    # Clean uninstaller
└── README.md
```

---

## Installation and Removal

### Install

```bash
# From GitHub (standard installation):
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/install.sh | bash

# From a local clone:
./install.sh
```

The installer copies files to:
- `~/.claude/skills/legal/` — main orchestrator
- `~/.claude/skills/legal-*/` — 13 sub-skills
- `~/.claude/agents/` — 5 agent markdown files
- `~/.claude/skills/legal/scripts/` — Python PDF script
- `~/.claude/skills/legal/templates/` — report template

It also checks for `python3` and `reportlab` (needed only for PDF generation).

### Python Dependency for PDF Generation

```bash
pip3 install reportlab
```

### Uninstall

```bash
./uninstall.sh
# or
curl -fsSL https://raw.githubusercontent.com/zubair-trabzada/ai-legal-claude/main/uninstall.sh | bash
```

---

## All 14 Commands

After installation, these slash commands are available inside Claude Code:

### Contract Analysis
| Command | Description |
|---------|-------------|
| `/legal review <file>` | **Flagship** — full review using 5 parallel subagents, produces scored report |
| `/legal risks <file>` | Deep risk analysis with 1-10 scoring and financial exposure estimates |
| `/legal compare <file1> <file2>` | Side-by-side diff of two contracts |
| `/legal plain <file>` | Translates every clause to plain English |
| `/legal negotiate <file>` | Generates counter-proposals with negotiation talking points |
| `/legal missing <file>` | Identifies protections that should exist but are absent |

### Document Generation
| Command | Description |
|---------|-------------|
| `/legal nda <description>` | Generates mutual, one-way, employee, or vendor NDA |
| `/legal terms <url>` | Generates terms of service based on website content |
| `/legal privacy <url>` | Generates GDPR/CCPA-compliant privacy policy from site scan |
| `/legal agreement <type>` | Generates freelancer contracts, SOWs, MSAs, partnerships, etc. |
| `/legal freelancer <file>` | Contractor-focused review flagging payment, IP, and kill-fee traps |

### Compliance and Reporting
| Command | Description |
|---------|-------------|
| `/legal compliance <url>` | Website compliance audit: GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, COPPA, SOC 2 |
| `/legal report-pdf` | Converts the most recent review to a professional PDF |

---

## Key Architecture: How `/legal review` Works

The flagship skill runs a 4-phase pipeline:

1. **Ingest** — reads the contract (file path, pasted text, or URL via WebFetch), classifies type (Service Agreement, NDA, Employment, SaaS, etc.), extracts metadata
2. **Parallel agents** — launches all 5 subagents simultaneously via the Agent tool, each receiving the full contract text
3. **Aggregate** — combines agent outputs into a weighted Contract Safety Score (0-100) with grade A+ through F
4. **Report** — writes `CONTRACT-REVIEW-[name]-[date].md` using the template in `templates/contract-review-template.md`

**Subagent weights in the final score:**
- `legal-risks.md` — 25% (highest; scores each clause 1-10)
- `legal-clauses.md` — 20% (clause extraction and categorization)
- `legal-compliance.md` — 20% (regulatory enforceability checks)
- `legal-recommendations.md` — 20% (generates counter-proposals)
- `legal-terms.md` — 15% (obligations/deadlines timeline)

---

## Key Files

### `legal/SKILL.md`
Main router. Defines all 14 commands, routing logic, input handling (file/text/URL), file naming conventions for output, tone/style rules, and the mandatory disclaimer that appears on every output.

### `agents/legal-risks.md`
The most complex agent. Defines 10 risk categories (Financial Exposure, Liability Transfer, Restrictive Covenants, Unclear Terms, etc.) and a 4-factor scoring formula:
- Severity × 0.40 + Likelihood × 0.25 + Financial Exposure × 0.20 + Asymmetry × 0.15
- Also runs a "poison pill" detection scan for hidden clauses

### `agents/legal-compliance.md`
Checks contracts against: GDPR Art. 28, CCPA/CPRA, state non-compete enforceability by jurisdiction (California is banned, Minnesota is banned, etc.), IRS independent contractor 20-factor test, usury laws by state, and consumer protection unconscionability doctrine.

### `agents/legal-recommendations.md`
Produces copy-ready alternative contract language and negotiation scripts. Uses a priority tier system (P0 Dealbreaker through P4 Cosmetic) and 8 recommendation types (Replace, Modify, Add, Delete, Carve-Out, Cap, Mutual, Clarify).

### `scripts/generate_legal_pdf.py`
Standalone CLI tool. Takes a JSON data file as input and produces a multi-section PDF using ReportLab. Includes a semi-circular score gauge graphic, risk bar chart, and full clause analysis.

**Usage:**
```bash
python3 scripts/generate_legal_pdf.py <json_data_file> [output_path]
```

The JSON input schema:
```json
{
  "score": 72,
  "grade": "B",
  "grade_label": "Fair",
  "executive_summary": "...",
  "details": {"type": "...", "parties": "...", "effective_date": "...", "term": "...", "total_value": "...", "governing_law": "..."},
  "risks": {"high": 3, "medium": 5, "low": 2, "high_clauses": "...", "medium_clauses": "...", "low_clauses": "..."},
  "clauses": [{"name": "...", "section": "...", "risk": "high|medium|low", "summary": "...", "risk_explanation": "...", "recommendation": "..."}],
  "negotiation_priorities": ["..."],
  "missing_protections": ["..."],
  "next_steps": ["..."]
}
```

### `skills/legal-report-pdf/SKILL.md`
AI skill that finds the most recent `CONTRACT-REVIEW-*.md` file, parses it, builds JSON data, and calls `generate_legal_pdf.py` or generates inline Python if the script is not found. Falls back to ReportLab inline generation when needed.

### `templates/contract-review-template.md`
Markdown template for the review output. Has placeholder blocks for every section: Contract Safety Score, Executive Summary, Contract Details table, Risk Dashboard, Clause-by-Clause Analysis (grouped by risk level), Missing Protections, Obligations & Deadlines, Compliance Flags, Negotiation Priorities, and Recommended Next Steps.

---

## Output File Naming Conventions

All generated files follow this pattern:

| Output Type | File Name |
|-------------|-----------|
| Contract review | `CONTRACT-REVIEW-[name]-[YYYY-MM-DD].md` |
| Freelancer review | `FREELANCER-REVIEW-[name]-[YYYY-MM-DD].md` |
| Contract comparison | `CONTRACT-COMPARISON-[YYYY-MM-DD].md` |
| NDA | `NDA-[Party1]-[Party2]-[YYYY-MM-DD].md` |
| Terms of service | `TERMS-OF-SERVICE-[company]-[YYYY-MM-DD].md` |
| Privacy policy | `PRIVACY-POLICY-[company]-[YYYY-MM-DD].md` |
| Compliance audit | `COMPLIANCE-AUDIT-[company]-[YYYY-MM-DD].md` |
| PDF report | `CONTRACT-REVIEW-REPORT.pdf` |

Files are written to the current working directory.

---

## Skill File Format

Each skill in `skills/*/SKILL.md` follows this pattern:
- Optional YAML front matter with `name`, `description`, `command` fields
- Plain English description of when the skill is invoked
- Numbered phases (Phase 1: Ingest, Phase 2: Analyze, Phase 3: Output, etc.)
- Detailed output format specification (exact markdown structure to produce)
- Error handling section

Agent files in `agents/*.md` follow the same pattern but are focused subagents, not user-facing commands.

---

## Contract Safety Score

| Score | Grade | Label | Recommendation |
|-------|-------|-------|----------------|
| 90-100 | A+ | Safe | Sign as-is |
| 80-89 | A | Good | Minor issues only |
| 70-79 | B | Fair | Some clauses need attention |
| 60-69 | C | Caution | Negotiate before signing |
| 40-59 | D | Risky | Strong negotiation needed |
| 0-39 | F | Dangerous | Do not sign without major revisions |

---

## Risk Indicator System

All skills use consistent risk indicators:
- `🔴 High Risk` — significant financial loss, legal liability, or loss of rights possible
- `🟡 Medium Risk` — moderate disadvantage or ambiguity
- `🟢 Low Risk / Standard` — acceptable or minor improvement possible

---

## Development Notes

### Testing with the Sample Contract

```bash
python3 generate_sample_contract.py
# Generates sample-contract.pdf for testing /legal review
```

### Testing PDF Generation

```bash
# Create a minimal test JSON:
echo '{"score": 72, "grade": "B", "grade_label": "Fair", "executive_summary": "Test.", "details": {}, "risks": {"high": 2, "medium": 3, "low": 1}}' > test.json
python3 scripts/generate_legal_pdf.py test.json output.pdf
```

### Modifying Skills

Each skill is a self-contained SKILL.md. To change a skill's behavior:
1. Edit the relevant file in `skills/<skill-name>/SKILL.md` or `agents/<agent-name>.md`
2. Re-run `./install.sh` from the repo root to push the updated file to `~/.claude/`

There is no build step — files are copied as-is by the installer.

### Adding a New Skill

1. Create `skills/new-skill-name/SKILL.md`
2. Add the skill name to the `SKILLS` array in `install.sh`
3. Add routing for the command in `legal/SKILL.md`
4. Add the removal path to `uninstall.sh`

---

## Notes for AI Assistants

- **Disclaimer is mandatory:** Every single output from every skill must include the legal disclaimer at the top. This is defined in `legal/SKILL.md` and repeated in individual agents.
- **No legal advice:** The system is designed to flag, analyze, and suggest — never to conclude that something is definitively legal or illegal. Always hedge with "likely," "may be," "consult an attorney."
- **5 parallel agents:** The `/legal review` skill is designed to launch all 5 subagents simultaneously. This is intentional for speed. The orchestrator in `skills/legal-review/SKILL.md` aggregates their independent outputs.
- **Input flexibility:** Skills that accept a `<file>` argument should handle file paths, pasted text, and URLs equally. The `legal/SKILL.md` orchestrator documents this explicitly.
- **PDF script vs. inline generation:** `skills/legal-report-pdf/SKILL.md` first searches for `generate_legal_pdf.py` in several locations before falling back to generating inline Python. If modifying PDF behavior, update both the script and the inline fallback.
- **Jurisdiction awareness:** `agents/legal-compliance.md` contains jurisdiction-specific non-compete tables and usury law tables. These have a knowledge cutoff date and should be flagged as potentially outdated when presented to users.
- **Contract type detection:** `skills/legal-review/SKILL.md` defines a detection table mapping contract signals (keywords like "deliverables," "salary," "SLA," etc.) to contract types and key risk areas. This calibrates the downstream agent analysis.
- **Scoring formula:** Risk scores in `agents/legal-risks.md` are calculated as `(Severity × 0.40) + (Likelihood × 0.25) + (Financial Exposure × 0.20) + (Asymmetry × 0.15)`. When in doubt, round up if financial exposure is uncapped.

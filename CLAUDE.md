# ai-legal-claude

A Claude Code skill suite for legal document analysis and generation. Provides 14 commands for contract review, risk assessment, compliance checking, and document generation — designed for freelancers and small businesses.

## Repository Structure

```
ai-legal-claude/
├── README.md                        # User-facing docs with command reference
├── install.sh                       # Installs all skills, agents, scripts, templates
├── uninstall.sh                     # Removes all installed components
├── generate_sample_contract.py      # Generates sample-contract.pdf for testing
├── sample-contract.pdf              # Intentionally flawed contract (test fixture)
├── legal/
│   └── SKILL.md                     # Main orchestrator — routes /legal commands
├── skills/                          # 13 specialized skills
│   ├── legal-review/SKILL.md        # Flagship: launches 5 parallel subagents
│   ├── legal-risks/SKILL.md         # Deep risk analysis (clause-by-clause)
│   ├── legal-compare/SKILL.md       # Side-by-side contract comparison
│   ├── legal-plain/SKILL.md         # Legalese → plain English translation
│   ├── legal-negotiate/SKILL.md     # Counter-proposal generator
│   ├── legal-missing/SKILL.md       # Missing protections finder
│   ├── legal-nda/SKILL.md           # Custom NDA generator
│   ├── legal-terms/SKILL.md         # Terms of service generator
│   ├── legal-privacy/SKILL.md       # Privacy policy generator
│   ├── legal-agreement/SKILL.md     # Business agreement templates
│   ├── legal-compliance/SKILL.md    # Compliance gap analysis
│   ├── legal-freelancer/SKILL.md    # Freelancer perspective review
│   └── legal-report-pdf/SKILL.md   # PDF report generator
├── agents/                          # 5 parallel subagents for /legal review
│   ├── legal-clauses.md             # Clause Analysis (20% weight)
│   ├── legal-risks.md               # Risk Assessment (25% weight — highest)
│   ├── legal-compliance.md          # Compliance Check (20% weight)
│   ├── legal-terms.md               # Terms & Obligations (15% weight)
│   └── legal-recommendations.md    # Recommendations (20% weight)
├── scripts/
│   └── generate_legal_pdf.py        # ReportLab PDF report generator
└── templates/
    └── contract-review-template.md  # Markdown template for review reports
```

## Command Reference

### Contract Analysis
| Command | Purpose |
|---------|---------|
| `/legal review` | Full analysis — launches 5 parallel subagents, produces Contract Safety Score |
| `/legal risks` | Deep risk analysis with clause-by-clause 1–10 scoring |
| `/legal compare` | Side-by-side diff of two contract versions |
| `/legal plain` | Translate legalese to plain English |
| `/legal negotiate` | Generate counter-proposals with specific replacement language |
| `/legal missing` | Identify protections that should exist but don't |

### Document Generation
| Command | Purpose |
|---------|---------|
| `/legal nda` | Generate custom NDA (mutual or one-way) |
| `/legal terms` | Generate Terms of Service (GDPR/CCPA compliant) |
| `/legal privacy` | Generate Privacy Policy |
| `/legal agreement` | Generate business agreements (freelancer, partnership, SOW, MSA) |
| `/legal freelancer` | Freelancer-perspective contract review |

### Compliance & Reporting
| Command | Purpose |
|---------|---------|
| `/legal compliance` | GDPR, CCPA, ADA, PCI-DSS, CAN-SPAM, SOC 2 audit |
| `/legal report-pdf` | Generate professional PDF report |

## Parallel Subagent Architecture (`/legal review`)

The flagship command launches 5 subagents simultaneously and aggregates their output:

```
/legal review
     │
     ├── legal-clauses    (20%) — identifies 20 clause types + 8 secondary flags
     ├── legal-risks      (25%) — scores 10 risk dimensions on 1–10 scale
     ├── legal-compliance (20%) — checks GDPR, CCPA, HIPAA, PCI-DSS, SOC 2, GLBA
     ├── legal-terms      (15%) — maps obligations, deadlines, financial exposure
     └── legal-recommendations (20%) — generates P0–P4 priority replacement language
                              │
                        Synthesized into:
                        CONTRACT-REVIEW report + Contract Safety Score (0–100)
```

**Contract Safety Score:**
- 90–100 = A+ Safe | 80–89 = A Good | 70–79 = B Fair | 60–69 = C Caution
- 40–59 = D Risky | 0–39 = F Dangerous

## Input Formats

All skills accept contracts as:
- File path: `/path/to/contract.pdf` or `/path/to/contract.md`
- Pasted text: paste directly into the prompt
- URL: Claude uses WebFetch to retrieve the document

## Output Conventions

- Outputs saved as Markdown files with timestamps
- Naming: `[COMMAND]-[party/topic]-[YYYY-MM-DD].md`
- Risk indicators: 🔴 high / 🟡 medium / 🟢 low
- Every output includes a legal disclaimer

## Installation

```bash
bash install.sh
```

Installs everything to `$HOME/.claude/skills` and `$HOME/.claude/agents`. Validates Python 3 and `reportlab` availability.

**Requirements:**
- Claude Code
- Python 3.8+
- `reportlab` library: `pip3 install reportlab`

To remove all components:
```bash
bash uninstall.sh
```

## PDF Report Generation

`scripts/generate_legal_pdf.py` uses ReportLab to produce professional reports with:
- Semi-circular score gauge
- Risk dashboard with color-coded tables
- Navy/blue color palette, red for danger, green for safety

Generate a test report:
```bash
python3 scripts/generate_legal_pdf.py
```

## Testing

Use `sample-contract.pdf` as a test fixture — it's intentionally flawed to exercise risk detection. Regenerate it with:
```bash
python3 generate_sample_contract.py
```

## Skill Development Conventions

- **SKILL.md frontmatter**: `name`, `description`, `command` fields required
- **Subagents**: defined as `.md` files in `agents/`, not `skills/`
- **Plain English alongside technical detail**: every skill explains concepts before instructions
- **Real-world impact quantification**: use concrete numbers ("$20,000–$100,000 in legal fees") not vague warnings
- **Decision matrices**: use tables to classify, prioritize, and route
- **No test suite, no CI** — validate manually with sample-contract.pdf

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`, `command`).
2. Add the command to `legal/SKILL.md`'s routing section.
3. Register the new skill in `install.sh` so it deploys to `$HOME/.claude/skills`.
4. Test against `sample-contract.pdf` before submitting.

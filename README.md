# Research with Receipts

A Claude Code plugin for structured, verified research. Every claim comes with a receipt.

## What This Does

Most AI research tools generate reports from scratch. This one does the opposite — it **extracts and verifies** information from documents you already have, ensuring every data point is traceable to a specific source.

**The problem:** LLMs hallucinate. Even RAG systems like NotebookLM, while significantly better, are not hallucination-free. When you're extracting data from annual reports, proxy statements, or research papers, a single fabricated data point can invalidate your entire analysis.

**The solution:** A structured 7-step workflow that separates extraction from verification, uses multi-round validation, and produces a Word document where every claim has a traceable source.

## How It Works

```
Step 1: Source Preparation    → Get documents into a local folder
Step 2: Import to NotebookLM  → Build a queryable knowledge base
Step 3: Define Dimensions     → Specify exactly what to extract
Step 4: Pre-flight Check      → Test one object to validate feasibility
Step 5: Batch Extraction      → Query NotebookLM for every object × dimension
Step 6: Multi-Round Verify    → Three rounds of independent verification
Step 7: Word Output           → Generate traceable report
```

### Multi-Round Verification

The core value. Three independent rounds catch different types of errors:

- **Round 1 (Self-audit):** Does the quoted source actually support the claim?
- **Round 2 (Re-query):** Ask the same question differently — do answers match?
- **Round 3 (Spot-check):** Random 20% sample — if any errors found, full re-check.

### Confidence Ratings

Every data point gets a status:

| Status | Meaning |
|--------|---------|
| ✅ | Direct evidence with source citation |
| ⚠️ | Indirect evidence — reasoning documented |
| ❌ | No data available in provided sources |

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Node.js >= 18
- A Google account (free tier works, 50 queries/day)
- Chrome browser (for NotebookLM authentication)

### Install the plugin

```bash
# Install NotebookLM MCP (required)
claude mcp add notebooklm npx notebooklm-mcp@latest

# Install python-docx (for Word output)
pip install python-docx

# Install this plugin
claude plugin add github:Kewanvk/research-with-receipts
```

### First-time setup

```bash
# Authenticate with NotebookLM (opens Chrome)
nlm login
```

## Usage

In Claude Code, trigger the skill with natural language:

```
"Help me extract data from these annual reports"
"I have some PDFs in ~/reports/, help me research them"
"Verify the claims in this document"
"Collect data from these proxy statements"
```

The skill will guide you through each step interactively.

## Why Not Just Use NotebookLM Directly?

NotebookLM is great at answering questions from your documents. But it doesn't:

- **Batch extract** across dozens of objects and dimensions automatically
- **Structure the output** into a standardized, traceable format
- **Multi-round verify** its own answers for consistency
- **Grade sources** or flag when evidence is indirect vs. direct
- **Refuse to guess** — it may still infer when it should say "no data"

This skill uses NotebookLM as the extraction engine, but wraps it in a disciplined research methodology.

## Built From Real Pain

This workflow was developed during a real academic research project involving 50+ board directors across 3 companies, extracting nationality, ethnicity, and education data from proxy statements. Along the way:

- 19 out of 27 source screenshots were invalid (anti-scraping pages, 404s)
- 8 directors had completely wrong education data
- Keyword matching was mistaken for semantic proof
- URLs were recorded but never verified as accessible

Every verification rule in this skill exists because a real error happened without it.

## Project Structure

```
research-with-receipts/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── research-with-receipts/
│       ├── SKILL.md                        # Core workflow (7 steps)
│       └── references/
│           ├── verification-rules.md       # Source grading + 8 error patterns
│           └── agent-workflow.md           # Multi-round verification details
├── README.md
└── LICENSE
```

## License

MIT

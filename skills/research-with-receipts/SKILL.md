---
name: Research with Receipts
description: >
  This skill should be used when the user asks to "extract data from documents",
  "verify information from reports", "research from annual reports", "fact-check
  these claims", "cross-verify this data", "help me collect data from these files",
  "organize research from PDFs", or needs structured, source-grounded data extraction
  with multi-round verification. Produces Word documents with full source traceability.
version: 0.1.0
---

# Research with Receipts

A structured research workflow that extracts data from user-provided documents, verifies every claim through multi-round validation via NotebookLM, and outputs a Word document where every data point has a traceable source. No guessing, no hallucinations — every claim comes with a receipt.

## Core Principle

**Verification over generation.** This skill does not generate research from scratch. It extracts and verifies information from documents the user already has. The goal is accuracy, not creativity.

## Prerequisites

- **NotebookLM MCP**: Install via `claude mcp add notebooklm npx notebooklm-mcp@latest`. Requires a Google account (free tier: 50 queries/day). Run `nlm login` for first-time authentication.
- **python-docx**: For Word document output. Install via `pip install python-docx`.

## Seven-Step Workflow

### Step 1: Source Preparation

Ensure all research materials are in a local folder.

**Two entry points:**
- **User provides URLs** → Download files to a local project folder (e.g., `~/research-[project-name]/sources/`)
- **User points to local files** → Confirm files exist and are readable

After preparation, list all source files and ask for confirmation: "These are all your source materials. Correct?"

### Step 2: Import to NotebookLM

1. Create a dedicated notebook via `notebook_create` with the project name
2. Import each local file via `source_add` (type=file for local files, type=url for web sources)
3. List all imported sources and confirm: "X files imported successfully. Ready to proceed."

### Step 3: Define Extraction Dimensions

Guide the user to clearly define what to extract. Ask:

1. **"What are the objects?"** — People, companies, products, etc.
2. **"What dimensions do you need for each object?"** — The specific data fields to extract
3. **"What counts as valid evidence for each dimension?"** — Acceptance criteria

Structure the answers into a clear specification:

```
Objects: [e.g., Board Directors]
Dimensions:
- [dimension_1]: [acceptance criteria]
- [dimension_2]: [acceptance criteria]
- ...
```

Confirm with the user before proceeding. Do NOT start extraction with vague dimensions.

### Step 4: Pre-flight Check (Single Object Test Run)

Before batch extraction, test with one object to validate feasibility.

1. Select the first object from the list
2. Query NotebookLM for every dimension (`notebook_query`)
3. Present results in a validation table:

| Dimension | Answer | Source Citation | Status |
|-----------|--------|----------------|--------|
| dim_1 | "exact quote..." | source_id: xxx | ✅ Direct evidence |
| dim_2 | no information found | — | ❌ Not available |
| dim_3 | partial match | source_id: yyy | ⚠️ Indirect / incomplete |

4. Ask: "These are the results for the first object. Should we adjust any dimensions before proceeding with all objects?"

### Step 5: Batch Extraction

For each object × each dimension:

1. Query NotebookLM with a precise, targeted question
2. Record four fields per data point:
   - **Answer**: The extracted information
   - **Source quote**: The exact passage NotebookLM cited
   - **Source**: source_id or document name
   - **Status**: ✅ Direct evidence / ⚠️ Indirect evidence / ❌ No data
3. For ❌ and ⚠️ items: record as-is. Do NOT fabricate, guess, or infer. Mark "No public source" when data is unavailable.

After extraction, present a summary table for user review.

### Step 6: Multi-Round Verification

Three rounds of verification to minimize errors. Refer to `references/verification-rules.md` for detailed rules.

**Round 1 — Self-audit:**
- For each ✅ item: Does the quoted passage actually support the conclusion?
- Check for common error patterns (see `references/verification-rules.md`):
  - Keyword match ≠ semantic match
  - Attribution errors (content from doc A attributed to doc B)
  - Number mix-ups across tables
- Downgrade problematic items (✅ → ⚠️) with notes

**Round 2 — Re-query with rephrased questions:**
- For all ⚠️ items: ask NotebookLM the same question in a different way
- Compare answers across rounds:
  - Consistent → increase confidence
  - Contradictory → flag as "source conflict"

**Round 3 — Spot-check:**
- Randomly sample 20% of ✅ items and re-verify
- If any errors found in sample → trigger full re-verification
- Log results: "Sampled X items, Y errors found, error rate Z%"

### Step 7: Word Document Output

Generate a Word document using python-docx with the following structure:

```
[Project Name] — Research Report
Generated: [date]
Methodology: Research with Receipts (multi-round verified extraction)

1. Source Materials
   - List of all imported documents/URLs

2. Extraction Specification
   - Objects and dimensions defined in Step 3

3. Data Table
   | Object | Dim_1 | Source | Dim_2 | Source | ... |
   (Every cell has a source reference)

4. Confidence Legend
   ✅ Direct evidence (with source quote)
   ⚠️ Indirect evidence (with reasoning)
   ❌ No data available

5. Verification Log
   - Round 1: X issues found, Y corrected
   - Round 2: Z items re-verified
   - Round 3: Sampled N items, error rate X%
```

Save to the project folder and provide the file path.

## Important Rules

1. **Never fabricate data.** If NotebookLM returns "no information found", record ❌. Do not fill gaps with general knowledge.
2. **Never use keyword matching as semantic proof.** A word appearing on a page does not mean the page proves what you need. Always check: Who is the subject? What is actually being stated?
3. **Source must be traceable.** Every data point must link to a specific document and passage. "Biographical sources" or "Inferred" are not valid source citations.
4. **Extraction and verification are separate steps.** The process that generates data must not be the same process that verifies it.
5. **Absence of evidence is a valid finding.** "No data" is an honest result. Never turn it into a guess.

## Additional Resources

### Reference Files

- **`references/verification-rules.md`** — Detailed verification rules, source grading system, and 8 common error patterns with real examples
- **`references/agent-workflow.md`** — Multi-round verification workflow details and sub-agent coordination patterns

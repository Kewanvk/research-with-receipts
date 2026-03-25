# Multi-Round Verification Workflow

## Overview

The verification workflow ensures accuracy through independent, sequential rounds of checking. Each round has a specific purpose and catches different types of errors.

## Round Details

### Round 1: Self-Audit (Content Accuracy)

**Purpose:** Catch errors in the extraction itself — wrong data, misattributed sources, semantic mismatches.

**Process:**
1. Take each ✅ (direct evidence) data point
2. Read the source quote that was recorded alongside it
3. Ask three questions:
   - Does this quote actually say what the data point claims?
   - Is the quote attributed to the correct source document?
   - Is the subject of the quote the same as the object being researched?
4. If any answer is "no" → downgrade to ⚠️ with a note explaining the issue

**Common catches in Round 1:**
- Quote says "operations in Colombia" but was used to prove nationality
- Quote is from Document A but was attributed to Document B
- Quote describes a different person with a similar name

### Round 2: Re-Query with Rephrased Questions (Consistency Check)

**Purpose:** Test whether the same information holds up when asked differently. Catches cases where the original question led to a biased or incomplete answer.

**Process:**
1. Take all ⚠️ items (and optionally a sample of ✅ items)
2. Formulate a different question that targets the same information
3. Query NotebookLM with the new question via `notebook_query`
4. Compare the two answers:

| Scenario | Action |
|----------|--------|
| Both answers agree, both cite sources | ✅ Upgrade confidence |
| Answers agree but second has no citation | ⚠️ Keep as indirect |
| Answers contradict each other | 🔴 Flag as "source conflict", investigate |
| Second query returns "no information" | ⚠️ Original may have been over-interpreted |

**Rephrasing strategies:**
- Change the question structure: "What is X's nationality?" → "Is X described as American anywhere in the sources?"
- Ask for the negative: "Is there any evidence that X is NOT American?"
- Ask for the source directly: "Which document mentions X's nationality?"

### Round 3: Spot-Check (Statistical Confidence)

**Purpose:** Estimate the overall error rate of the dataset and decide whether full re-verification is needed.

**Process:**
1. From all ✅ items, randomly select 20% (minimum 3 items)
2. For each selected item, re-query NotebookLM and verify independently
3. Calculate error rate:
   - 0 errors → Dataset is likely reliable. Proceed to output.
   - 1+ errors → Full re-verification required for all items.

**Logging:**
Record the spot-check results in the verification log:
```
Round 3 Spot-Check:
- Total ✅ items: [N]
- Sampled: [X] items ([Y]%)
- Errors found: [Z]
- Error rate: [Z/X * 100]%
- Decision: [Proceed / Full re-verification]
```

## Sub-Agent Coordination

For large datasets (10+ objects × 3+ dimensions), use sub-agents to parallelize verification.

### Recommended Pattern

```
Main Agent
├── Extraction Phase
│   └── Main agent handles all extraction sequentially
│       (to maintain consistent questioning strategy)
│
├── Verification Phase (parallelizable)
│   ├── Sub-Agent 1: Audit objects 1-5
│   ├── Sub-Agent 2: Audit objects 6-10
│   └── Sub-Agent 3: Audit objects 11-15
│       Each sub-agent:
│       1. Reads the extraction results for assigned objects
│       2. Runs Round 1 (self-audit) independently
│       3. Runs Round 2 (re-query) for flagged items
│       4. Returns: corrected data + issue log
│
└── Consolidation Phase
    └── Main agent merges all sub-agent results
        1. Resolve any conflicts between sub-agents
        2. Run Round 3 (spot-check) on merged dataset
        3. Generate final output
```

### Key Rules for Sub-Agents

1. **Each sub-agent gets a clear, bounded scope** — specific objects and dimensions, not "check everything"
2. **Sub-agents do not modify the extraction data directly** — they return findings and corrections, main agent applies them
3. **Sub-agents must have access to NotebookLM** — they need `notebook_query` for Round 2
4. **Main agent handles the final spot-check** — Round 3 must be done on the merged dataset, not per sub-agent

## NotebookLM Query Best Practices

### Question Design

**Good questions (specific, verifiable):**
- "Does the Wikipedia article for Linda Hudson describe her as 'American businesswoman'?"
- "According to the BofA 2023 proxy Board Diversity Matrix, what is Sharon Allen's self-identified race/ethnicity?"
- "What does NNDB say about Lionel Nowell III's nationality?"

**Bad questions (vague, invites inference):**
- "Tell me about Linda Hudson's background"
- "What is Sharon Allen's ethnicity?"
- "Is Lionel Nowell American?"

### Handling "No Information" Responses

When NotebookLM responds with "the provided sources do not contain this information":

1. Record ❌ immediately — do not try to infer the answer
2. Note which sources were searched (the notebook's source list)
3. If the user needs this data, they must provide additional source documents
4. In 1.0, do NOT go searching external sources. Record "No data in provided sources" and move on.

### Rate Limit Awareness

Free Google accounts have ~50 queries/day. For large projects:

1. Plan queries efficiently — batch related questions
2. Avoid redundant queries — check if a previous query already answered the question
3. If approaching the limit, inform the user and suggest:
   - Continuing the next day
   - Prioritizing the most critical dimensions first

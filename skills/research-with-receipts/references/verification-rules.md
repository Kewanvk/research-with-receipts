# Verification Rules

## Source Grading System

Every source must be graded before its evidence is accepted. Higher-grade sources can be used directly; lower-grade sources require cross-verification.

| Grade | Source Type | Examples | Usage Rule |
|-------|-----------|----------|------------|
| A | Primary official documents | Annual reports, proxy statements, government filings, SEC filings | Use directly |
| B | Official websites / self-declarations | Company website bios, LinkedIn (self-written), personal statements | Use directly |
| C | Third-party databases | NNDB, Prabook, Encyclopedia.com, Bloomberg profiles | Cross-verify with one other source |
| D | News / secondary reporting | Media articles, Wikipedia, press releases | Cross-verify with two other sources |
| ✗ | Inference / assumption | Guessing ethnicity from surname, nationality from appearance | Never use. Mark "No public source" |

## Citation Specificity Rules

Not all NotebookLM responses are equally reliable. Assess each response:

| Response Type | Example | Verdict |
|--------------|---------|---------|
| Exact quote with source_id | "Linda Parker Hudson is an American businesswoman" (source: c5c0934b) | ✅ Accept |
| Paraphrased with source reference | "According to the document, Hudson is American" | ⚠️ Acceptable, but note it's paraphrased |
| Vague attribution | "Based on the sources, she appears to be American" | ⚠️ Needs re-query with more specific question |
| No source cited | "She is likely American based on her background" | ❌ Reject. This is inference, not evidence |
| Explicit absence | "The provided sources do not contain this information" | ❌ Record as "No data". Do NOT fill the gap |

## Eight Common Error Patterns

These are real error patterns observed in production research tasks. Review this list before starting any verification round.

### 1. Keyword Match ≠ Semantic Match

Finding a word on a page does not mean the page proves what is needed. "Colombia" appearing in a business description ("operations in Colombia") is not the same as "born in Colombia."

**Prevention:** For every piece of evidence, ask: "Who is the subject of this sentence? Is it describing this specific person/object's own attribute?"

### 2. Automating Judgment Tasks

Some tasks require semantic understanding and cannot be delegated to keyword matching, regex, or OCR scripts. A script can find where a word appears, but cannot judge whether that occurrence is relevant evidence.

**Prevention:** Distinguish between mechanical operations (can automate) and judgment operations (must be done semantically). Never automate the judgment layer.

### 3. Filling Data from Memory Instead of Source

Writing data based on what "feels right" without opening the source to confirm. Example: writing "BS Computer Science" when the actual degree was "Symbolic Systems."

**Prevention:** Every data point must be copy-pasted from the source, never typed from memory. If you cannot point to the exact source text, you do not have evidence.

### 4. Inferring Identity from Surface Features

Assuming ethnicity from surnames, nationality from appearance, or identity from indirect signals.

**Prevention:** Without an explicit source, mark "No public source." Never use surface features to fill identity fields. An empty cell is better than a wrong one.

### 5. Overconfident Visual Judgment

Making definitive claims about image content when resolution is limited or details are unclear. Stating "confirmed correct" when the honest assessment is "I cannot read this clearly."

**Prevention:** Be honest about confidence levels for visual content. If something is unclear, state "Cannot verify visually" rather than guessing.

### 6. Non-Traceable Source Citations

Writing vague source labels like "Biographical sources" or "Inferred — no explicit text" that appear to fill the source field but provide no way for a reader to verify.

**Prevention:** A valid source is a URL, document name + page number, or specific database entry. If none exists, write "No public source" and state the reasoning separately.

### 7. Unverified URLs

Recording source URLs without confirming they are accessible. Some URLs return 404, contain non-ASCII encoding errors, or redirect to unrelated content.

**Prevention:** Collection and verification are two separate steps. Every URL must be confirmed accessible and content-matching before inclusion in the final output.

### 8. Superficial Verification

Designing verification checks around "what is easy to check" rather than "what is most likely to be wrong." Checking formatting and ordering while skipping the most error-prone content (whether the evidence actually supports the claim).

**Prevention:** Design verification items starting from "where are errors most likely?" not "what is easiest to check?" The most important check is always: does this evidence actually prove this specific claim?

## Verification Checklist Template

Before finalizing any research output, run through this checklist:

```
□ Every data point has a specific, traceable source (not "inferred" or "biographical sources")
□ Every source URL has been verified accessible
□ No data was filled from memory — all copy-pasted from source
□ Keyword matches have been checked for semantic validity
□ Identity fields (nationality, ethnicity) have explicit source declarations, not inferences
□ Visual judgments are marked with confidence levels
□ Extraction was done by a separate process from verification
□ Verification checks target the most error-prone items, not the easiest to check
```

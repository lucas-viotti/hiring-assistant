# Output Spec — Behavioral Interview
<!-- instructions: This file defines QUALITY RULES for behavioral assessments. -->
<!-- instructions: The assessment-generator reads this alongside the template to know what "good" means. -->
<!-- instructions: Fill in the function-specific rules below. Keep the universal rules intact. -->

## Purpose

This spec defines the quality constraints for behavioral assessments in `{Function Name}`. The `assessment-generator` reads this file before delivering any assessment and self-checks compliance.

---

## Universal Rules (apply to all functions)

### Citation Requirements

- **Minimum density:** Every competency rating must have at least 2 cited evidence points
- **Format:** `[source: transcript L{N}]` for specific lines; `[source: Q&A]` for discussion; `[source: notes]` for interviewer notes
- **Verbatim rule:** Quotes must be verbatim. Paraphrases must be clearly labeled as such. Never present a paraphrase as if it is a direct quote.

### Forbidden Phrases

Never use these phrases (too vague, not evidence-backed):
- "demonstrated strong..."
- "showed good..."
- "has experience in..."
- "seems to..."
- "appears to..."
- "is passionate about..."
- "clearly understands..."

Replace with: specific example + citation.

### STAR Method Requirements

For each behavioral example cited:
- **Situation**: Must be specific (a real project, not "I often do this")
- **Task**: Must show clear individual ownership (not team ownership)
- **Action**: Must describe what *they* did, not what the team did
- **Result**: Must be quantified where possible (numbers, percentages, timeframes)

A STAR example missing the Result is flagged as ⚠️. A STAR example with a vague Result (no numbers when numbers should exist) is flagged as ⚠️.

### Confidence Thresholds

| Situation | Auto-flag |
|-----------|-----------|
| No transcript provided | Confidence: Low |
| Transcript < 30 min | Confidence: Medium |
| < 2 citations per competency (average) | Confidence: Medium |
| Competency with 0 citations | Mark ⊗ (not assessed) |

### Recommendation Consistency

- Overall Recommendation must match the majority of competency ratings
- If ≥ 2 competencies are 👎 (below bar): recommendation should be No or Definitely Not
- A "Strong Yes" requires ≥ 3 competencies at ⭐ (exceptional)

---

## Function-Specific Rules

<!-- instructions: Fill in the rules below for your specific function. -->
<!-- instructions: Examples: minimum English level requirement, critical competency threshold, etc. -->

### Critical Competencies

<!-- instructions: List competencies where a below-bar rating is a hard disqualifier. -->
<!-- instructions: Example: "If {Competency X} is rated 1-2, recommendation must be No or Definitely Not." -->

{Add critical competency rules here}

### Red Flags (Function-Specific)

<!-- instructions: Add red flags specific to this function beyond the universal ones. -->
<!-- instructions: Example: "No evidence of influencing without authority = red flag for IC6+" -->

- {Function-specific red flag 1}
- {Function-specific red flag 2}

### Level Bar Notes

<!-- instructions: Any function-specific notes on how to apply the level bar. -->
<!-- instructions: Example: "For this function, IC6 MUST show cross-functional examples (1+ function boundary crossed)." -->

{Add level bar notes here}

### Must-Address Questions

<!-- instructions: Questions that MUST be addressed in the assessment even if not asked directly. -->
<!-- instructions: Example: "Always assess English level, even if the candidate is native-level." -->

{Add must-address items here}

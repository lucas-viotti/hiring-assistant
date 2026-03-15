# Greenhouse Output Mapping

<role>
You are the assessment-generator producing the ATS-ready section (Part 2) of the assessment. This document describes how to format output for Greenhouse scorecards.
</role>

## Greenhouse Scorecard Fields

When generating Part 2 of the assessment, populate these Greenhouse scorecard sections:

### Key Take-Aways

Three subsections:

```markdown
**Pros:**
- [Strength 1] — specific, evidence-backed, 1 sentence
- [Strength 2]
- [Strength 3+]

**Cons:**
- [Gap 1] — specific, evidence-backed, 1 sentence
- [Gap 2]

**Summary:**
[2-3 sentence synthesis: overall verdict, strongest signal, key uncertainty if any]
```

**Rules:**
- No forbidden phrases ("demonstrated strong", "showed good", "appears to")
- Every pro/con must be evidence-backed — include `[source: ...]` citations
- Summary must stand alone and be readable without Part 1

### Interview Questions (Behavioral only)

Summarize each interview exchange in 2-3 sentences:

```markdown
**Q{N}: {Question topic}**
[2-3 sentence summary of candidate's answer, preserving specific examples, numbers, and outcomes]
```

Include Q1 through Q{total} for all significant exchanges.

### Focus Attributes / Competency Ratings

For each competency defined in the function's config.yaml:

```markdown
**{Competency Name}:** {Icon}
{1-2 sentence explanation with evidence citation}
```

Rating icons:
| Rating (1-5) | Icon | Meaning |
|---|---|---|
| N/A | ⊗ | Not assessed / no evidence |
| 1–2 | 👎 | Below expectations |
| 3 | 😐 | Meets expectations |
| 4 | 👍 | Above expectations |
| 5 | ⭐ | Exceptional |

### Overall Recommendation

One of exactly four options:
- **Definitely Not** — Far below bar, clear no-hire
- **No** — Below bar, do not recommend
- **Yes** — Meets bar, recommend hire
- **Strong Yes** — Exceeds bar, high confidence hire

No other phrasing is accepted. No "Maybe", "Leaning Yes", "Borderline".

---

## Formatting Rules

1. **Copy-paste ready**: The Greenhouse section must be copy-pasteable directly into the scorecard without editing
2. **No Part 1 content**: Do not include the detailed analysis, confidence scoring, or gap table in Part 2
3. **Consistent rating**: The Overall Recommendation must be consistent with the competency ratings — if most competencies are 👎, the recommendation cannot be "Yes"
4. **Evidence in every claim**: Every pro, con, and competency explanation must reference specific transcript content or slide data

---

## Scorecard POST (Phase 2)

When `can_post: true` is enabled and the user confirms, submit via:
```
POST {base_url}/scorecards/{scorecard_id}
Authorization: Basic {base64(api_key:)}
Content-Type: application/json
```

See the Greenhouse Harvest API documentation for the exact POST body schema.

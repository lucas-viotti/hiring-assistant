# Output Schema

This document defines what every **assessment output** MUST contain.

---

## Purpose

All assessments must follow a consistent structure to ensure:
- Complete evaluation coverage
- Easy Greenhouse submission
- Quality validation

---

## Document Structure

Every assessment has **TWO PARTS**:

```
┌─────────────────────────────────────────────────────────────────┐
│  PART 1: DETAILED ANALYSIS                                      │
│  For AI reasoning and interviewer review                        │
│  ├── Header                                                     │
│  ├── Final Assessment                                           │
│  ├── Confidence Score                                           │
│  ├── Guidelines Compliance                                      │
│  ├── Gap Analysis Table                                         │
│  ├── Red Flag Alerts                                            │
│  ├── Competency Analysis                                        │
│  ├── Structured Reasoning Chain                                 │
│  └── Cohort Calibration                                         │
├─────────────────────────────────────────────────────────────────┤
│  PART 2: GREENHOUSE-READY                                       │
│  Copy-paste directly into Greenhouse scorecard                  │
│  ├── Key Take-Aways                                             │
│  ├── Competency Ratings (with icons)                            │
│  └── Overall Recommendation                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 1: Required Sections

### 1. Header

| Field | Required | Example |
|-------|----------|---------|
| Candidate Name | Yes | Maria Santos |
| Level | Yes | IC6 |
| Date | Yes | 2026-01-04 |
| Interviewer(s) | Yes | [Interviewer Name] |
| Interview Type | Yes | Business Case |

### 2. Final Assessment

**One of exactly:**
- **Yes** — Meets bar, recommend hire
- **No** — Below bar, do not recommend

> Never use: "Maybe", "Leaning Yes", "Borderline"

### 3. Confidence Score

| Level | Criteria |
|-------|----------|
| **High** | 4+ citations per competency, clear evidence, no inference |
| **Medium** | 2-3 citations, some context inference, minor gaps |
| **Low** | <2 citations, significant inference, major gaps |

Include reasoning table explaining confidence level.

### 4. Guidelines Compliance

**Required table format:**

```markdown
## Guidelines Compliance Check

**Guideline Source:** `guidelines/[function]/[type]/`

| Guideline Requirement | Met? | Evidence |
|----------------------|------|----------|
| [Requirement 1] | ✅/⚠️/❌ | [source] |
| [Requirement 2] | ✅/⚠️/❌ | [source] |
```

### 5. Gap Analysis Table

**Required format:**

```markdown
| IC[X] Expectation | Required | Candidate Showed | Status |
|-------------------|----------|------------------|--------|
| [Expectation 1] | [Bar] | [Evidence] | ✅/⚠️/❌ |
| [Expectation 2] | [Bar] | [Evidence] | ✅/⚠️/❌ |
```

**Status key:**
- ✅ Meets or exceeds
- ⚠️ Partially meets
- ❌ Does not meet

### 6. Red Flag Alerts

**Required format:**

```markdown
## Red Flag Alerts

**Detected:**
- [ ] [Flag]: [Evidence with source]

**Checked — Not Present:**
- [x] [Flag 1]
- [x] [Flag 2]
```

### 7. Competency Analysis

For each competency, include:
- Rating (1-5)
- Evidence with citations
- Comparison to level bar

### 8. Structured Reasoning Chain

**Required format:**

```markdown
## Structured Reasoning Chain

### Evidence Summary
**For Yes:** [List strongest points with sources]
**For No:** [List concerns with sources]

### Level Bar Check
- Meets bar on: [X/N competencies]
- Below bar on: [List]
- Critical showstoppers: [Yes/No]

### Final Reasoning
> Given [evidence], compared to [level bar], recommendation is **[Yes/No]** because [synthesis].
```

### 9. Cohort Calibration

Compare to recent candidates at same level (if available).

---

## Part 2: Greenhouse Section

### Key Take-Aways

**Required format:**

```markdown
## Key Take-Aways

**Pros:**
- [Strength 1 with evidence]
- [Strength 2 with evidence]

**Cons:**
- [Gap 1]
- [Gap 2]

**Summary:**
[2-3 sentence synthesis]
```

### Competency Ratings

**Rating icons:**

| AI Rating | Icon | Meaning |
|-----------|------|---------|
| N/A | ⊗ | Not assessed |
| 1-2 | 👎 | Below expectations |
| 3 | 😐 | Meets expectations |
| 4 | 👍 | Above expectations |
| 5 | ⭐ | Exceptional |

**Required format per competency:**

```markdown
### [Competency Name]
**Rating:** [Icon]
**Explanation:** [1-2 sentences with evidence]
```

### Overall Recommendation

**One of exactly:**
- **Definitely Not** — Far below bar
- **No** — Below bar
- **Yes** — Meets bar
- **Strong Yes** — Exceeds bar

---

## Forbidden Phrases

These vague statements are NEVER allowed:

| Forbidden | Replace With |
|-----------|--------------|
| "demonstrated strong..." | Specific example with source |
| "showed good..." | Specific example with source |
| "has experience in..." | Specific achievement with source |
| "seems to..." | Evidence-backed statement |
| "appears to..." | Evidence-backed statement |

---

## Citation Rules

Every claim MUST have a source:

| Source Type | Format |
|-------------|--------|
| Slide | `[source: slide ##]` |
| Transcript | `[source: transcript]` |
| Specific line | `[source: transcript L##]` |
| Q&A | `[source: Q&A]` |
| Notes | `[source: notes]` |

---

## Validation Checklist

Before delivering, verify:

**Structure**
- [ ] All required sections present
- [ ] PART 1 and PART 2 separated
- [ ] Header complete

**Evidence**
- [ ] Every claim has citation
- [ ] No forbidden phrases
- [ ] Quantified outcomes where available

**Greenhouse Section**
- [ ] All competencies rated with icons
- [ ] Key Take-Aways has Pros/Cons/Summary
- [ ] Recommendation uses exact wording

**Consistency**
- [ ] Recommendation matches evidence
- [ ] Confidence reflects evidence density
- [ ] Level bar correctly applied


# {Candidate Name} — Business Case Assessment
<!-- instructions: Template for business case detailed analysis (Part 1). -->
<!-- instructions: Quality rules are in output-spec-business-case.md — keep them separate. -->

**Level:** {IC Level}
**Date:** {YYYY-MM-DD}
**Interviewer:** {Name}
**Function:** {Function Name}
**Case Variant:** {Case Name}

---

## ⚡ Final Assessment

**Recommendation:** {Yes / No}
**Confidence:** {High / Medium / Low}

---

## Source Materials

<!-- instructions: List all materials used in this evaluation. -->

| File | Type | Status |
|------|------|--------|
| {Filename}.pdf | Presentation | Extracted |
| transcript.md | Transcript | {Available/Not available} |
| notes.md | Interviewer notes | {Available/Not available} |
| {Guidelines doc} | Guidelines | Read |

---

## Confidence Score

| Dimension | Assessment |
|-----------|------------|
| Materials completeness | {all slides readable / partial / text-only} |
| Transcript/Q&A available | {yes/no} |
| Must-identify insights coverage | {N of M identified} |
| Inference required | {minimal/moderate/significant} |

---

## Guidelines Compliance Check

<!-- instructions: The must-identify insights come from the guidelines document — load them dynamically, never hardcode. -->

**Guideline Source:** `~/.hiring-assistant/guidelines/{function}/business-case/{case-id}/`

| Must-Identify Insight | Importance | Identified? | Evidence |
|-----------------------|------------|-------------|----------|
| {Insight 1 from guidelines} | Critical | ✅/⚠️/❌ | [source: slide {N}] |
| {Insight 2} | Important | ✅/⚠️/❌ | |
| {Insight 3} | Nice-to-have | ✅/⚠️/❌ | |

---

## Gap Analysis

| IC{X} Expectation | Required | Candidate Showed | Status |
|-------------------|----------|------------------|--------|
| {Expectation 1} | {Bar} | {Evidence} | ✅/⚠️/❌ |
| {Expectation 2} | | | |

---

## Red Flag Alerts

**Detected:**
- [ ] {Flag if present}: [source: slide {N}]

**Checked — Not Present:**
- [x] Missed all critical insights
- [x] No data-driven recommendations
- [x] Proposed solution ignores key constraints from case data
- [x] Recommendations not tied to analysis
<!-- Add function-specific red flags from output-spec-business-case.md -->

---

## Competency Analysis

<!-- instructions: One section per competency from the guidelines document. -->
<!-- instructions: For business case: cite slide numbers and Q&A exchanges, not just transcript lines. -->

### {Competency 1 Name}

**Rating:** {1–5}

**Evidence from materials:**
- {Specific finding from presentation} [source: slide {N}]
- {Finding from Q&A} [source: Q&A]

**Assessment:** {Comparison to IC{X} bar.}

---

### {Competency 2 Name}
<!-- Repeat for each competency -->

---

## Structured Reasoning Chain

### Evidence Summary

**For Yes:**
- {Strongest finding} [source: slide {N}]

**For No:**
- {Key gap} [source: ...]

### Level Bar Check

- Meets bar on: {N}/{M} competencies
- Below bar on: {list}
- Critical insights missed: {Yes/No — list if Yes}

### Final Reasoning

> Given {key evidence}, compared to the IC{X} bar, recommendation is **{Yes/No}** because {synthesis}.

---

## Cohort Calibration

**Cohort context (IC{X}, {case} case, last {N} assessments):**
- Average Yes rate: {X}%
- This candidate's insight identification: {above/at/below} typical

---

---

# PART 2: GREENHOUSE-READY

## Skills and Knowledge Ratings

<!-- instructions: List the competencies your Greenhouse scorecard tracks for business cases. -->
<!-- instructions: Icon scale: ⊗ N/A | 👎 Below | 😐 Meets | 👍 Above | ⭐ Exceptional -->

**{Competency 1 Name}:** {icon}
{1-2 sentence explanation with evidence}

**{Competency 2 Name}:** {icon}
{explanation}

---

## Key Take-Aways

**Pros:**
- {Strength 1 — evidence-backed}
- {Strength 2}

**Cons:**
- {Gap 1}
- {Gap 2}

**Summary:**
{2-3 sentence synthesis. Standalone-readable. Lead with recommendation signal.}

---

## Overall Recommendation

**{Definitely Not / No / Yes / Strong Yes}**

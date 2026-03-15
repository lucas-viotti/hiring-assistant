# Output Spec — Business Case Interview
<!-- instructions: Quality rules for business case assessments. assessment-generator reads this alongside the template. -->

## Universal Rules (apply to all functions)

### Citation Requirements

- Presentation slides: `[source: slide {N}]`
- Q&A discussion: `[source: Q&A]`
- Interviewer notes: `[source: notes]`
- Every competency rating must have at least 2 cited evidence points

### Forbidden Phrases

- "demonstrated strong analytical skills" → cite specific analysis performed
- "showed good business sense" → cite specific recommendation and rationale
- "appears to understand" → cite evidence or mark as ⊗

### Must-Identify Insights Compliance

- All **critical** insights from guidelines must appear in the assessment
- **Important** insights: note if missed (but not a hard disqualifier)
- **Nice-to-have**: note but never penalize for missing

### Confidence Thresholds

| Situation | Auto-flag |
|-----------|-----------|
| No materials provided | Cannot proceed |
| Only Q&A, no presentation | Confidence: Medium |
| Presentation text-only (no images processed) | Note: "⚠️ Visual data not analyzed" |

---

## Function-Specific Rules

<!-- instructions: Fill in function-specific rules below. -->

### Critical Insights Threshold

<!-- instructions: Load threshold rules from the guidelines document at runtime. Do not hardcode. -->
<!-- instructions: The guidelines document contains a "must-identify insights" section with Importance levels. -->
<!-- instructions: Apply the following default logic unless the guidelines override it: -->

- **IC4/IC5**: Candidate must identify at least 50% of Critical insights for a Yes recommendation.
- **IC6**: Candidate must identify all Critical insights for a Yes recommendation.
- **IC7+**: Candidate must identify all Critical insights AND at least half of Important insights; must show quantified trade-offs.

If the guidelines document specifies different thresholds, those override the defaults above.

### Case-Specific Red Flags

<!-- instructions: Load case-specific red flags from the guidelines document at runtime. Do not hardcode. -->
<!-- instructions: Guidelines will contain a "red flags" or "disqualifiers" section per case variant. -->
<!-- instructions: The following are universal red flags that apply regardless of case: -->

- Candidate proposes a solution that directly contradicts a hard constraint stated in the case data → 🔴 Critical Gap
- Candidate's recommendations have no connection to the analysis they presented → 🔴 Critical Gap
- Candidate shows no awareness of business or financial impact → ⚠️ Concern
- Candidate blames stakeholders or the org for problems without proposing solutions → ⚠️ Concern

### Level Bar Notes

<!-- instructions: Load level-specific bars from the guidelines at runtime. Apply these defaults if not overridden: -->

- **IC4/IC5**: Describes the problem correctly; proposes a plausible solution; basic data interpretation
- **IC6**: Identifies root causes; quantifies impact; proposes solution with trade-offs and prioritization
- **IC7+**: Frames the problem at a strategic level; proactively considers cross-functional and regulatory implications; drives toward a decision with confidence

### Probing Questions Protocol

<!-- instructions: If candidate misses a critical insight during the presentation, the interviewer may probe. -->
<!-- instructions: Weight probed answers as follows (unless guidelines specify otherwise): -->

- **Unprompted finding**: Full credit — demonstrates independent analytical thinking
- **Finding after light probe** (e.g., "What else did you notice?"): 80% credit
- **Finding after direct probe** (e.g., "What do you think about X metric?"): 50% credit — candidate knew when asked, but didn't surface independently
- **Still missed after direct probe**: 0% credit — note as Critical Gap for Critical insights

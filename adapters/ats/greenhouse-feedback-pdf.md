# Greenhouse Interview Feedback PDF Parser

Instructions for extracting structured scorecard data from Greenhouse's "Interview Feedback" PDF export.

## What this PDF is

The Greenhouse Interview Feedback PDF is exported by Talent Acquisition partners and shared with bar raisers before a debrief. It contains ALL submitted scorecards for a candidate, concatenated into a single document.

## PDF Structure

The PDF has two major sections. The first is noise; the second is signal.

### Section 1: Application Metadata (SKIP)

The first 30-50% of the PDF is application metadata — custom fields, demographic questions, intern program questions from multiple countries, empty fields. This section:

- Starts at page 1 with the candidate name, position, and location
- Contains fields like "Personal Info", "Other Details", custom questions
- Every page ends with "Powered by Greenhouse"
- Has many fields with value `--` (empty)

**Skip everything until you reach the "Scorecards for {Candidate Name}" header.**

### Section 2: Scorecards (EXTRACT)

The scorecard section starts with:

```
Scorecards for {Candidate Name}
Overall Recommendations
```

#### Overall Recommendations Block

Immediately after the header, there is a list of all interviewers with their recommendations:

```
{Interviewer Name}

{Recommendation}

{Stage Name}
```

This repeats for each interviewer. Extract all of these into a summary table.

Possible recommendation values:
- `Yes`
- `No`
- `Strong Yes`
- `Definitely Not`

Possible stage names:
- `Screening`
- `Exercise Presentation` (business case)
- `Technical Interview` (behavioral)
- `Organization Values Interview`
- `Hiring Manager Interview`
- `Debriefing`
- Other custom stage names

#### Scorecard Summary Block

After the recommendations, there is a "Scorecard Summary" section with aggregated competency comments across interviewers. This contains:

```
{Competency Category}

{Competency Name}
{Interviewer Name}:

{Comment text}
```

Extract each competency comment attributed to its interviewer.

Common competency categories:
- `Organization Values` — with specific value names as sub-competencies
- `Skills and Knowledge (What)` — with competency names like "Written Communication", "Analytical Thinking", "Problem Solving", "Strategic Thinking", "Time Management", "Critical Reasoning"
- `Focus Attributes` — behavioral competencies

#### Individual Scorecards

After the summary, each interviewer's full scorecard appears:

```
{Stage Name}
Interviewed by {Interviewer Name}

{Date and time}
{Recommendation}

Key Take-Aways
{Free-text content — this is the most valuable field}

Question 1 *

{Question title or topic}

{Answer/notes}

Question 2 *
...

Skills and Knowledge (What)

{Competency Name}
{Rating icon or text}
{Comment}
```

For each individual scorecard, extract:
1. **Interviewer name**
2. **Stage name**
3. **Date**
4. **Recommendation** (Yes/No/Strong Yes/Definitely Not)
5. **Key Take-Aways** — the free-text block (most important field)
6. **Per-competency ratings and comments**
7. **Question responses** (if filled — many will say "Interviewer did not answer this question")

## Extraction Rules

1. **"Interviewer did not answer this question"** — skip these entirely, do not include in output
2. **"Powered by Greenhouse"** — page footer, ignore
3. **Duplicate content** — the Scorecard Summary and Individual Scorecards often contain the same comments. Prefer the Individual Scorecard version (more complete context).
4. **Rating extraction** — ratings may appear as:
   - Numeric: `1`, `2`, `3`, `4`, `5`
   - Descriptive: `Mixed`, `Yes`, `No`, `Strong Yes`, `Definitely Not`
   - Icons: `👎`, `😐`, `👍`, `⭐` (rare in PDF export)
   - Stars: `★★★★☆` or similar
   - If no explicit rating but a comment exists, mark as "commented, no rating"
5. **Key Take-Aways parsing** — look for `Pros:` / `Cons:` / `Summary:` subsections within the free text. If not structured, treat the entire block as a single narrative.
6. **Level comments** — watch for interviewers mentioning a different level than the candidate is applying for (e.g., "No for IC7 but Strong Yes at IC6"). Flag these explicitly — they are critical for debrief discussion.

## Output Format

Return structured data per interviewer:

```xml
<scorecard>
  <interviewer>{name}</interviewer>
  <stage>{stage name}</stage>
  <date>{date}</date>
  <recommendation>{Yes|No|Strong Yes|Definitely Not}</recommendation>
  <level_comment>{any explicit level commentary, or empty}</level_comment>
  <key_takeaways>
    <pros>{bullet points}</pros>
    <cons>{bullet points}</cons>
    <summary>{summary text}</summary>
  </key_takeaways>
  <competencies>
    <competency>
      <name>{competency name}</name>
      <rating>{numeric or descriptive}</rating>
      <comment>{interviewer's comment}</comment>
    </competency>
    <!-- repeat for each rated competency -->
  </competencies>
  <questions>
    <question>
      <number>{N}</number>
      <topic>{question title}</topic>
      <response>{interviewer's response}</response>
    </question>
    <!-- only include questions that were actually answered -->
  </questions>
</scorecard>
```

## Common Issues

| Issue | Handling |
|-------|---------|
| PDF text extraction scrambles column layout | Greenhouse PDFs are single-column; this is rare. If it happens, rely on the "Interviewed by" headers to separate scorecards |
| Interviewer appears multiple times | Normal — one entry in Overall Recommendations per stage. An interviewer doing Screening + Debriefing will have 2 entries |
| Competency names vary between stages | Expected — behavioral uses "Focus Attributes", business case uses "Skills and Knowledge (What)". Map by competency name, not category |
| Debrief scorecard entry | The Debriefing stage scorecard (often from TA) contains the debrief outcome. Flag this separately — it is the historical record, not an interviewer assessment |

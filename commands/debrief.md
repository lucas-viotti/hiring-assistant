# /hiring-debrief

Prepares interviewers and bar raisers for a candidate debrief meeting. Two modes serve two distinct personas with different data needs.

**Usage:** `/hiring-debrief [--mode interviewer|bar-raiser]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first

---

## Step 1 — Mode Selection

Ask **first** (unless `--mode` flag is provided):

```
What is your role in this debrief?

[1] Interviewer — I participated in 1 or more stages and want to
    prepare my talking points for the meeting
[2] Bar raiser — I'm facilitating the debrief and need to see all
    interviewers' scorecards to build the discussion agenda
```

If `--mode` flag provided → skip mode selection.

**This must happen before any data loading**, because the mode determines what data is needed and from where.

---

## Step 2 — Candidate Identification

Ask (unless `--auto` or a candidate name is inferred from recent context):

```
Candidate name:
```

If ambiguous → list matching candidate directories and ask user to choose.

---

## Path A — Interviewer Mode

### Step 3A — Load Your Assessments

Glob: `~/.hiring-assistant/candidates/{candidate_name}/assessment-*.md`

List found assessment files and their modification dates. These are only the assessments **you** generated via `/hiring-evaluate`.

If none found:
```
No assessments found for {candidate_name}.
Run /hiring-evaluate for the stage(s) you participated in first.
```
Stop.

### Step 4A — Generate Prep Sheet

Dispatch: `agents/debrief-synthesizer.md`

```xml
<candidate_name>{name}</candidate_name>
<mode>interviewer</mode>
<assessment_files>
  {list of found local assessment file paths}
</assessment_files>
```

### Step 5A — Output

Print your debrief prep sheet. Offer to save:
```
Save to ~/.hiring-assistant/candidates/{name}/debrief-prep.md? [Yes / No]
```

---

## Path B — Bar Raiser Mode

### Step 3B — Data Source Selection

The bar raiser needs ALL interviewers' scorecards — not just their own. Since each interviewer's assessment lives on their own machine, the data must come from a shared source.

Ask:
```
How should I get the scorecards?

[1] Greenhouse API — auto-pull all submitted scorecards
    (requires GREENHOUSE_API_KEY)
[2] Interview Feedback PDF — the export from Greenhouse that
    TA typically shares before the debrief
[3] Local assessment files — if interviewers shared their
    assessment files with you directly
```

#### Source 1: Greenhouse API

Check: `GREENHOUSE_API_KEY` environment variable.

If set:
- Ask for the Greenhouse application URL or ID (check if available from Step 2 context)
- Fetch: `GET {base_url}/applications/{application_id}/scorecards`
- Parse each scorecard using the field mappings in `adapters/ats/greenhouse.yaml`
- Present summary: "Found {N} scorecards from {interviewer names}. Proceed? [Yes / No]"

If not set:
```
GREENHOUSE_API_KEY not found. Options:
- Set it (see /hiring-help for instructions) and try again
- Choose option [2] PDF or [3] local files instead
```
Go back to source selection.

#### Source 2: Interview Feedback PDF

Ask:
```
Provide the path to the Greenhouse Interview Feedback PDF:
(This is the PDF export that TA shares before the debrief)
```

Read the PDF. Parse using the instructions in `adapters/ats/greenhouse-feedback-pdf.md`.

Present summary: "Extracted {N} scorecards from {interviewer names}. Proceed? [Yes / No]"

#### Source 3: Local Assessment Files

Glob: `~/.hiring-assistant/candidates/{candidate_name}/assessment-*.md`

If none found → suggest source [1] or [2] instead.

If found → list files and confirm.

### Step 4B — Cross-Interviewer Analysis

Dispatch: `agents/debrief-synthesizer.md`

```xml
<candidate_name>{name}</candidate_name>
<mode>bar-raiser</mode>
<data_source>{greenhouse-api | greenhouse-pdf | local}</data_source>
<scorecards>
  {structured scorecard data from the selected source}
</scorecards>
```

### Step 5B — Output

Print the debrief agenda. Offer to save:
```
Save agenda to ~/.hiring-assistant/candidates/{name}/debrief-agenda.md? [Yes / No]
```

### Step 6B — Post-Meeting Verdict (optional)

After the debrief meeting, the bar raiser can return to record the outcome:

```
/hiring-debrief --mode bar-raiser
> {candidate name}
> Record verdict
```

Or run with the verdict flag:
```
/hiring-debrief --mode bar-raiser --verdict
```

This triggers the synthesis sub-mode of the debrief-synthesizer agent:
- Asks for the final recommendation and key decisive factors
- Saves to `~/.hiring-assistant/candidates/{name}/debrief-synthesis.md`

---

## Flags

| Flag | Effect |
|------|--------|
| `--mode interviewer` | Go directly to interviewer path |
| `--mode bar-raiser` | Go directly to bar raiser path |
| `--verdict` | Bar raiser only: skip agenda, go directly to recording post-meeting verdict |
| `--auto` | Use most recently active candidate; skip confirmations |

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| No local assessments (interviewer mode) | Direct user to run `/hiring-evaluate` for their stage(s) |
| No scorecards found (bar-raiser, API) | Suggest PDF or local file source instead |
| PDF parsing fails or is empty | Ask user to verify it's a Greenhouse Interview Feedback export; suggest API or local source |
| Only one interviewer's scorecard available | Generate with single-interviewer data; note limitation |
| Candidate name ambiguous | List matching candidate directories and ask user to choose |
| Greenhouse API key missing | Show setup instructions; suggest PDF or local source |

# /hiring-evaluate

Orchestrates the post-interview assessment pipeline. Runs the 8-gate flow defined in `core/workflow.md`.

**Usage:** `/hiring-evaluate [--auto | --confirm]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first
2. **At least one function configured:** `config.functions` is non-empty

---

## Gate Sequence

### GATE 1 — Candidate Discovery

Dispatch: `agents/interview-router.md`

```xml
<config>{~/.hiring-assistant/config.yaml}</config>
<search_window>30</search_window>
```

Receive: `<routing_result>` with candidate, level, function, type, greenhouse_url.

---

### GATE 2 — Case Identification (business-case only)

If `routing_result.interview_type == "business-case"`:

Read `~/.hiring-assistant/functions/{function}/config.yaml` → find active cases.

- If exactly one active case → auto-select
- If multiple active cases → ask user:
  ```
  Which case is this candidate presenting?
  [1] {Case 1 name}
  [2] {Case 2 name}
  ```

Note selected `case_id` in routing context.

---

### GATE 3 — Materials Loading (business-case only)

If `routing_result.interview_type == "business-case"`:

Dispatch: `agents/material-reader.md`

```xml
<candidate_name>{from routing_result}</candidate_name>
<case_id>{selected case_id from Gate 2}</case_id>
<function>{from routing_result}</function>
```

Receive: `<materials_result>` with status, page count, extracted content.

If `materials_result.status == "missing"`:
  Ask: "Please provide the PDF path for {Candidate Name}'s business case presentation:"

---

### GATE 4 — Guidelines

Dispatch: `agents/guideline-loader.md`

```xml
<routing_result>{from Gate 1}</routing_result>
<config>{~/.hiring-assistant/config.yaml}</config>
<org_config>{~/.hiring-assistant/org-config.yaml if present}</org_config>
```

Receive: `<guideline_load_result>` with guidelines content and function config.

**If guideline_load_result.guidelines_status = "missing" AND user chose not to proceed:**
Stop. Show access request instructions.

---

### GATE 4b — Re-evaluation Detection

Check: `~/.hiring-assistant/candidates/{candidate_name}/assessment-{interview_type}.md`

If file exists:
```
An existing assessment was found for {Candidate Name} (behavioral).
Modified: {file modification date}

Options:
[1] Update the existing assessment (re-run evaluation)
[2] View the existing assessment without re-running
[3] Create a new assessment (keep both)
```

If `--auto` flag: re-run if file is > 24h old; otherwise reuse.

---

### GATE 5 — Transcript

Dispatch: `agents/transcript-fetcher.md`

```xml
<routing_result>{from Gate 1}</routing_result>
<config>{~/.hiring-assistant/config.yaml}</config>
```

Receive: `<transcript>` with content, or `status="none"`.

---

### GATE 5b — Supplementary Notes

Check: `~/.hiring-assistant/candidates/{candidate_name}/notes.md`

If file exists:
- Read the file
- Note: "Supplementary notes found for {candidate_name}. They will be included in the evaluation."

---

### GATE 7 — Evaluation

Dispatch: `agents/assessment-generator.md`

```xml
<routing_result>{from Gate 1}</routing_result>
<guideline_load_result>{from Gate 4}</guideline_load_result>
<transcript>{from Gate 5}</transcript>
<notes>{from Gate 5b, or empty}</notes>
<materials_result>{from Gate 3 if business-case, otherwise empty}</materials_result>
<calibration_examples>{loaded from ~/.hiring-assistant/functions/{function}/examples/{level}/ if present}</calibration_examples>
```

Receive: `<assessment_result>` with status, path, recommendation, confidence.

---

### GATE 8 — Finalize

**If `--auto` flag is NOT set:**
Show the full assessment to the user and ask:
```
Assessment complete. Review above and confirm:
[Confirm] — save to cohort and finish
[Revise] — go back and update
```

Wait for confirmation.

**If `--auto` flag is set:**
Proceed immediately.

Dispatch: `agents/cohort-updater.md`

```xml
<assessment_result>{from Gate 7}</assessment_result>
<routing_result>{from Gate 1}</routing_result>
<config>{~/.hiring-assistant/config.yaml}</config>
```

Receive: `<cohort_update_result>` with paths and Greenhouse link.

---

## Completion

```
✅ Evaluation complete for {Candidate Name} ({Level} {Interview Type})

Assessment: ~/.hiring-assistant/candidates/{name}/assessment-{interview_type}.md
Cohort: Updated

{Greenhouse link if available}

Next steps:
- Open Greenhouse and paste Part 2 into your scorecard
- Run /hiring-debrief prep-interviewer before the debrief meeting
```

---

## Flags

| Flag | Effect |
|------|--------|
| `--auto` | Skip all confirmations; auto-pick most recent candidate; reuse files if < 24h old; skip review step |
| `--confirm` | Pause at every gate and ask before proceeding (useful for first evaluations) |

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| Config not found | "Run /hiring-setup first." |
| No calendar match | Falls back to manual entry in interview-router |
| Guidelines missing | Shows access request instructions; offers limited evaluation |
| No transcript | Proceeds with notes-only (confidence: Low) or stops if user declines |
| Assessment-generator fails | Show partial results; ask user how to proceed |

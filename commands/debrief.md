# /hiring-debrief

Synthesizes all assessments for a candidate. In `prep-interviewer` mode, produces a structured debrief agenda with talking points per disagreement. In `synthesis` mode, produces a consolidated verdict after the debrief discussion.

**Usage:** `/hiring-debrief [--mode prep-interviewer|synthesis]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first

---

## Step 1 — Candidate Identification

Ask (unless `--auto` or a candidate name is inferred from recent context):
```
Which candidate?
Candidate name:

Which mode?
[1] prep-interviewer — generate a debrief agenda before the meeting
[2] synthesis — record the final consolidated verdict after discussion
```

If `--mode` flag provided → skip mode selection.

---

## Step 2 — Locate Assessments

Glob: `~/.hiring-assistant/candidates/{candidate_name}/assessment-*.md`

List all found assessment files and their modification dates.

If none found:
```
No assessments found for {candidate_name}.
Run /hiring-evaluate first, then come back.
```
Stop.

---

## Step 3 — Synthesis

Dispatch: `agents/debrief-synthesizer.md`

```xml
<candidate_name>{name}</candidate_name>
<mode>{prep-interviewer or synthesis}</mode>
<assessment_files>
  {list of found assessment file paths}
</assessment_files>
```

---

## Step 4 — Output

**prep-interviewer mode:**
- Print debrief agenda to screen
- Offer to save: "Save agenda to `~/.hiring-assistant/candidates/{name}/debrief-agenda.md`? [Yes / No]"

**synthesis mode:**
- Print consolidated verdict
- Confirm: "`~/.hiring-assistant/candidates/{name}/debrief-synthesis.md` saved."

---

## Flags

| Flag | Effect |
|------|--------|
| `--mode prep-interviewer` | Go directly to agenda generation |
| `--mode synthesis` | Go directly to synthesis mode |
| `--auto` | Use most recently active candidate; skip confirmations |

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| No assessments found | Direct user to run /hiring-evaluate |
| Only one stage available | Generate with single-stage data; note limitation |
| Candidate name ambiguous | List matching candidate directories and ask user to choose |

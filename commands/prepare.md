# /hiring-prepare

Generates a pre-interview briefing for a candidate — candidate background, suggested probing questions per competency, and level bar reminder.

**Usage:** `/hiring-prepare [--auto]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first
2. **At least one function configured:** `config.functions` is non-empty

---

## Step 1 — Candidate Discovery

Dispatch: `agents/interview-router.md`

```xml
<config>{~/.hiring-assistant/config.yaml}</config>
<search_window>30</search_window>
```

Receive: `<routing_result>` with candidate, level, function, type.

---

## Step 2 — Guidelines

Dispatch: `agents/guideline-loader.md`

```xml
<routing_result>{from Step 1}</routing_result>
<config>{~/.hiring-assistant/config.yaml}</config>
<org_config>{~/.hiring-assistant/org-config.yaml if present}</org_config>
```

Receive: `<guideline_load_result>` with guidelines and function config.

---

## Step 2b — Materials Acquisition

Acquire candidate materials (resume for behavioral, submitted presentation for business case) to enrich the briefing.

### Path A — Greenhouse API available (`GREENHOUSE_API_KEY` is set)

1. Extract application ID from `routing_result.greenhouse_url`
2. Fetch: `GET /v1/applications/{application_id}/attachments`
3. Filter by interview type:
   - **Behavioral**: download attachments of type `resume`, `cover_letter`
   - **Business case**: download attachments of type `take_home_test` (the submitted presentation)
4. Save to `~/.hiring-assistant/candidates/{candidate_name}/materials/`
5. Dispatch: `agents/material-reader.md` to extract content
6. Pass extracted `<materials_result>` to Step 3

### Path B — No API key (fallback)

1. Search Gmail for the Greenhouse interview reminder:
   - Query: `from:no-reply@greenhouse.io subject:"interview" "{candidate_name}"`
   - OR: `from:no-reply@greenhouse.io "View the interview kit" "{candidate_name}"`
2. Extract the "View the interview kit" URL from the email body

**If interview kit URL found:**
```
Found Greenhouse interview kit for {Candidate Name}:
{interview_kit_url}

Please download the candidate's {resume | presentation} from the kit
and provide the file path below:

File path:
```

**If no email found:**
```
No Greenhouse interview kit found in your recent email.

To include candidate materials in the briefing:
1. Open Greenhouse → find {Candidate Name} → download their {resume | presentation}
2. Provide the file path below

Or type "skip" to prepare without materials.

File path:
```

3. If user provides a path → dispatch `agents/material-reader.md` to extract content
4. If user types "skip" → proceed without materials (briefing notes "Materials: Not available")

Pass `<materials_result>` (or empty) to Step 3.

---

## Step 3 — Briefing

Dispatch: `agents/prepare-briefing.md`

```xml
<routing_result>{from Step 1}</routing_result>
<guideline_load_result>{from Step 2}</guideline_load_result>
<materials_result>{from Step 2b, or empty if skipped}</materials_result>
```

Receive: completed briefing markdown.

---

## Step 4 — Output

Print the full briefing to screen.

Offer to save:
```
Briefing ready for {Candidate Name} ({Level} {Interview Type}).
Save to ~/.hiring-assistant/candidates/{name}/briefing-{type}.md? [Yes / No]
```

---

## Flags

| Flag | Effect |
|------|--------|
| `--auto` | Skip confirmations; auto-pick most recent candidate; save briefing without asking |

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| Config not found | "Run /hiring-setup first." |
| No calendar match | Falls back to manual entry in interview-router |
| Guidelines missing | Generates briefing with competency structure only (no level bar or must-identify insights) |
| Greenhouse API unavailable | Falls back to Gmail search for interview kit URL |
| No candidate materials found | Proceeds without materials; briefing notes "Materials: Not available" |

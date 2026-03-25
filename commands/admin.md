# /hiring-admin

Admin command for function owners and power users. Provides four subcommands: validate, clean, sync, and new-function.

**Usage:** `/hiring-admin [validate|clean|sync|new-function]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first

---

## Subcommand Selection

If no subcommand is given, show the menu:

```
/hiring-admin — Admin Tools

[1] validate      — Check plugin structure and config files
[2] clean         — Scan candidate files for retention window expiry
[3] sync          — Pull latest guidelines from configured repos
[4] new-function  — Bootstrap a new function (guided wizard)

Which would you like to run?
```

---

## Subcommand: validate

Run: `tests/validate-plugin.sh` (structure check only)

Show full output. On success:
```
✅ Plugin structure valid.
```

On failure:
```
❌ Validation failed. See output above.
```

---

## Subcommand: clean

Scan `~/.hiring-assistant/candidates/` for files older than the configured `retention_days`.

Read `retention_days` from `~/.hiring-assistant/config.yaml` (or `org-config.yaml`).

If `retention_days == 0`:
```
Retention cleanup is disabled (retention_days: 0). No action taken.
```

Otherwise, list all candidate directories and their most recent file modification dates.

Flag directories where all files are older than `retention_days` days:

```
The following candidate directories are past the retention window ({N} days):

  ~/.hiring-assistant/candidates/jane-doe/
    Last modified: {date} ({X} days ago)
    Files: assessment-behavioral.md, transcript.md

  ~/.hiring-assistant/candidates/john-smith/
    Last modified: {date} ({X} days ago)
    Files: assessment-business-case.md

Delete these directories? [Yes / No / Review each]
```

If user confirms → delete flagged directories and confirm.

---

## Subcommand: sync

Read `guidelines_repo` from `~/.hiring-assistant/functions/{function}/config.yaml` for each configured function.

For each function with a guidelines repo configured:
- Check if `~/.hiring-assistant/guidelines/{function}/` exists and is a git repo
- If yes → run `git -C ~/.hiring-assistant/guidelines/{function}/ pull`
- Show pull output

```
Syncing guidelines for sales-engineering...
  Already up to date.  (or: N commits pulled)

Sync complete.
```

If a guidelines directory is not a git repo:
```
⚠️ ~/.hiring-assistant/guidelines/sales-engineering/ is not a git repository.
Re-run /hiring-setup to reconfigure guidelines access.
```

---

## Subcommand: new-function

Guided wizard for hiring leads bootstrapping a new function. Creates function config, generates templates, and optionally pushes to the org config repo.

### Step 1 — Function Identity

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  New Function Setup
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

This wizard creates a complete function configuration with templates
ready for evaluation. You can always edit the generated files later.

Function ID (lowercase-hyphenated, e.g., "bco", "product-ops"):
```

Store as `function_id`.

```
Display name (e.g., "Business Consulting & Operations"):
```

Store as `function_name`.

Validate: function ID must be lowercase alphanumeric with hyphens only. Check that `~/.hiring-assistant/functions/{function_id}/` does not already exist. If it does:
```
A function with ID "{function_id}" already exists.
  Config: ~/.hiring-assistant/functions/{function_id}/config.yaml

[1] Overwrite (regenerate all files)
[2] Cancel
```

### Step 2 — Interview Types

```
Which interview types does this function use?

[1] Behavioral only
[2] Business case only
[3] Both behavioral and business case
```

Store as `interview_types` (list of: `behavioral`, `business-case`).

### Step 3 — Competencies

**If behavioral selected:**
```
List the behavioral competencies to evaluate.
Enter one per line. Empty line to finish.

These come from your function's evaluation guidelines. Common examples:
  Problem Structuring, Communication, Stakeholder Management,
  Business Sense, Result Oriented

Competency 1:
Competency 2:
...
(empty line to finish)
```

Store as `behavioral_competencies` (list of `{name}`). Generate IDs by lowercasing and hyphenating.

**If business case selected:**
```
List the business case competencies to evaluate.
Enter one per line. Empty line to finish.

Competency 1:
...
(empty line to finish)
```

Store as `business_case_competencies`.

### Step 4 — Levels

```
Which levels does this function hire for? (comma-separated)

Available: IC4, IC5, IC6, IC7, M2, M3

Levels:
```

Store as `levels` (list). Validate each entry is one of: IC4, IC5, IC6, IC7, M2, M3.

### Step 5 — Greenhouse Scorecard (Optional)

```
Do you have a Greenhouse scorecard screenshot or PDF for this function?

The wizard can read it to auto-extract:
  - Section names and question prompts
  - Rating fields and competency labels
  - Scorecard layout for the ATS-ready template

[1] Yes — I'll provide a file
[2] No — I'll configure the scorecard manually later
```

**If user provides a file:**

1. Read the provided image or PDF using the appropriate adapter (material-reader agent with pdf/image adapter)
2. Extract:
   - Section names (e.g., "Interview Questions", "Focus Attributes", "Key Take-Aways")
   - Question prompts (e.g., "Q1: Initial Introductions", "Q2: Deep Dive")
   - Rating fields with competency labels
   - Overall recommendation format
3. Confirm with user:
   ```
   I extracted the following from your scorecard:

   Sections:
     1. Interview Questions — {N} questions found
        Q1: {topic}
        Q2: {topic}
        ...
     2. Focus Attributes — {N} competencies
        {list}
     3. Attributes — {N} competencies
        {list}
     4. Key Take-Aways (Pros / Cons / Summary)
     5. Overall Recommendation (Definitely Not / No / Yes / Strong Yes)

   Is this correct? [Yes / Edit / Skip scorecard]
   ```
4. Store as `greenhouse_sections`.

**If skipped or no file:**
- Use a default scorecard structure with the competencies from Step 3
- Mark as `greenhouse.configured: false` in config

### Step 6 — Generate Files

Read scaffold templates from `{plugin-dir}/templates/function/` and substitute collected values.

**Substitution rules:**
- `{function-id}` → `function_id`
- `{Function Display Name}` → `function_name`
- `{competency-N-id}` → generated ID (lowercase, hyphenated) from competency name
- `{Competency N Name}` → competency name as entered by user
- Levels list → from Step 4
- Cases → from Step 3 (if business case selected)
- Greenhouse sections → from Step 5 (if provided)

**Files generated:**

1. `~/.hiring-assistant/functions/{function_id}/config.yaml`
   - From `templates/function/config.yaml` scaffold
   - Fill: function, name, status ("active"), interview_types, levels, competencies, greenhouse

2. `~/.hiring-assistant/functions/{function_id}/templates/assessment-detailed-behavioral.md` *(if behavioral)*
   - From `templates/function/templates/assessment-detailed-behavioral.md`
   - Substitute competency names into rating table rows and STAR analysis sections

3. `~/.hiring-assistant/functions/{function_id}/templates/assessment-ats-behavioral.md` *(if behavioral)*
   - From `templates/function/templates/assessment-ats-behavioral.md`
   - Substitute competency names into Focus Attributes and Attributes sections
   - If Greenhouse scorecard was provided: match section layout exactly

4. `~/.hiring-assistant/functions/{function_id}/templates/output-spec-behavioral.md` *(if behavioral)*
   - From `templates/function/templates/output-spec-behavioral.md`
   - Substitute competency names in compliance check and rating sections

5. `~/.hiring-assistant/functions/{function_id}/templates/prep-briefing-behavioral.md` *(if behavioral)*
   - From `templates/function/templates/prep-briefing-behavioral.md`
   - Substitute competency names in "Competencies to Probe" section

6–9. Same four templates for `*-business-case.md` *(if business case selected)*

Show:
```
✓ Created ~/.hiring-assistant/functions/{function_id}/config.yaml
✓ Created ~/.hiring-assistant/functions/{function_id}/templates/assessment-detailed-behavioral.md
✓ Created ~/.hiring-assistant/functions/{function_id}/templates/assessment-ats-behavioral.md
✓ Created ~/.hiring-assistant/functions/{function_id}/templates/output-spec-behavioral.md
✓ Created ~/.hiring-assistant/functions/{function_id}/templates/prep-briefing-behavioral.md

Function "{function_name}" is ready.
  Status: active
  Interview types: {types}
  Levels: {levels}
  Competencies: {N behavioral}, {M business case}
```

### Step 7 — Guidelines

```
Does a private guidelines repo already exist for this function?

[1] Yes — I'll provide the repo name
[2] No — I'll paste guidelines files manually
[3] No — skip for now, I'll set up guidelines later
```

**If [1]:**
```
Guidelines repo name (without org prefix, e.g., "hiring-guidelines-bco"):
```
- Update `config.yaml` with `guidelines_repo: "{provided_name}"`
- Attempt to clone: `gh repo clone {org}/{provided_name} ~/.hiring-assistant/guidelines/{function_id}/`
- If clone succeeds: `✓ Guidelines cloned`
- If clone fails: show access request instructions, offer paste fallback

**If [2]:**
```
Please provide the path to your guidelines file(s):
```
- Copy to `~/.hiring-assistant/guidelines/{function_id}/`
- Show: `✓ Guidelines copied to ~/.hiring-assistant/guidelines/{function_id}/`

**If [3]:**
```
No problem. The function is configured but evaluations will require guidelines.
When ready, either:
  - Create a guidelines repo and run /hiring-admin sync
  - Or paste files with: /hiring-admin new-function (re-run, select existing function)
```

### Step 8 — Push to Org Repo (Optional)

```
Push this function config to the org repo? [Yes / No]
(Requires write access to {org}/{content_repo})
```

**If Yes:**
1. Read `content.repo` from `~/.hiring-assistant/org-config.yaml`
2. Clone the org repo to a temp directory:
   ```bash
   temp_dir=$(mktemp -d)
   gh repo clone {org}/{content_repo} "$temp_dir"
   ```
3. Copy function files:
   ```
   ~/.hiring-assistant/functions/{function_id}/config.yaml → $temp_dir/functions/{function_id}/config.yaml
   ~/.hiring-assistant/functions/{function_id}/templates/  → $temp_dir/functions/{function_id}/templates/
   ```
4. Commit and push:
   ```bash
   cd "$temp_dir"
   git add functions/{function_id}/
   git commit -m "feat: add {function_name} function config and templates"
   git push origin main
   ```
5. Clean up: `rm -rf "$temp_dir"`
6. Show:
   ```
   ✓ Pushed functions/{function_id}/ to {org}/{content_repo}
   Other interviewers can now run /hiring-setup to get this function.
   ```

**If push fails (no write access):**
```
Push failed — you may not have write access to {org}/{content_repo}.
The function is configured locally and ready to use.

To share with your org, ask a repo admin to:
  1. Add you as a contributor to {org}/{content_repo}
  2. Or submit the files manually via PR
```

**If No:**
```
Function configured locally only. To share later, re-run:
  /hiring-admin new-function
and select the existing function to push.
```

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| validate-plugin.sh not found | "Run from the hiring-assistant plugin directory." |
| No candidate directories | "No candidate data found at ~/.hiring-assistant/candidates/" |
| Git pull fails (auth) | Show git error; suggest `gh auth status` to check credentials |
| Config not found | "Run /hiring-setup first." |
| new-function: function already exists | Offer overwrite or cancel |
| new-function: scorecard read fails | Skip scorecard; use default structure with provided competencies |
| new-function: push to org fails | Keep local config; show instructions for manual PR |

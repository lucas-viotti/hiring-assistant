# /hiring-setup

Onboarding wizard. Detects your identity, validates access, and writes `~/.hiring-assistant/config.yaml`.

**Usage:** `/hiring-setup`

Run once before using any other command. Re-run any time to update your configuration.

---

## Pre-flight

Check if `~/.hiring-assistant/config.yaml` already exists.

If it exists:
```
An existing configuration was found.
Modified: {file modification date}

Options:
[1] Re-run setup (update configuration)
[2] Cancel
```

If user selects [2]: Stop.

---

## Step 1 — Identify You

Use the Google Workspace MCP tool `people_getMe` to retrieve the current user's identity.

Extract:
- Display name
- Primary email address

If `people_getMe` fails or is unavailable:
- Ask: "What is your name and work email address?"
- Use the provided values

Show:
```
Setting up hiring-assistant for: {Display Name} ({email})
```

---

## Step 2 — Detect Functions from Calendar

Use `calendar_listEvents` to search the past 30 days for interview-related events.

Parameters:
```
timeMin: {30 days ago}
timeMax: {today}
q: "interview"
maxResults: 50
```

Parse event titles to detect function keywords:
- "Sales Engineering", "SE", "Sales Eng" → `sales-engineering`
- "Business Case", "BCO", "Business Consulting" → `bco`
- "PM", "Product Manager", "Product Management" → `product-manager`
- "Design", "UX", "Product Designer" → `design`

Show detected functions:
```
Detected from your calendar:
  ✓ sales-engineering (found {N} interviews in the past 30 days)

Additional functions available: (configure manually later with /hiring-admin)
```

Ask:
```
Is this correct? [Yes / No — let me choose manually]
```

If "No": show a text prompt to enter function IDs manually (comma-separated).

Store the confirmed list as `detected_functions`.

---

## Step 2b — Bootstrap Function Directories

For each function in `detected_functions`, check if the function config already exists:

```
~/.hiring-assistant/functions/{function}/config.yaml
```

**If it already exists:** Skip — show `✓ {function} — config already exists`.

**If it does NOT exist:** Run the interactive function setup below.

### Interactive function config creation

Show:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Setting up: {function}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

I need a few details to configure this function. You can always edit
the config later at ~/.hiring-assistant/functions/{function}/config.yaml
```

**Q1 — Display name:**
```
Function display name (e.g. "Sales Engineering"):
```

**Q2 — Interview types:**
```
Which interview types does this function use?

[1] Behavioral only
[2] Business case only
[3] Both behavioral and business case
```

**Q3 — Levels:**
```
Which levels do you interview for? (comma-separated, e.g. IC4,IC5,IC6,IC7):
```

**Q4 — Competencies (behavioral):** *(only if behavioral selected)*
```
List the behavioral competencies evaluated in this function.
Enter one per line, or type "skip" to fill in later.

Examples: Stakeholder Management, Problem Solving, Fast Learner

Competency 1:
Competency 2:
...
(empty line to finish)
```

**Q5 — Cases (business case):** *(only if business case selected)*
```
What is the name of the current business case?
(e.g. "Strategic Analysis", "System Scaling")

Case name:
```

**Q6 — Competencies (business case):** *(only if business case selected)*
```
List the business case competencies evaluated in this function.
Enter one per line, or type "skip" to fill in later.

Competency 1:
...
(empty line to finish)
```

### Create the function directory

1. Create directories:
   ```
   ~/.hiring-assistant/functions/{function}/
   ~/.hiring-assistant/functions/{function}/templates/
   ~/.hiring-assistant/functions/{function}/examples/
   ```

2. Read the scaffold template from `{plugin-dir}/templates/function/config.yaml`

3. Fill in the collected values:
   - `function` → function ID
   - `name` → display name from Q1
   - `status` → `"active"` (if competencies were provided) or `"draft"` (if skipped)
   - `interview_types` → based on Q2 selection
   - `levels` → from Q3
   - `competencies` → from Q4/Q6 (generate IDs by lowercasing and hyphenating)
   - `cases` → from Q5 (generate ID by lowercasing and hyphenating)

4. Write the filled config to `~/.hiring-assistant/functions/{function}/config.yaml`

5. Copy default assessment templates from `{plugin-dir}/templates/function/templates/` to `~/.hiring-assistant/functions/{function}/templates/`

Show:
```
✓ Created function config: ~/.hiring-assistant/functions/{function}/config.yaml
  Status: {active | draft — fill in competencies to activate}
  Interview types: {behavioral, business-case}
  Levels: {IC4, IC5, IC6, IC7}
  Competencies: {N} configured {or "0 — fill in later to activate"}
```

If status is "draft":
```
⚠ Function is in draft status because competencies were skipped.
  Edit ~/.hiring-assistant/functions/{function}/config.yaml to add them,
  then change status to "active".
```

---

## Step 3 — Clone Org Config + Acquire Guidelines

### Pre-check — GitHub CLI

Run:
```bash
gh auth status 2>&1
```

**Case A — gh not installed:**
If command is not found:
```
GitHub CLI (gh) is not installed or not in your PATH.

Org configs and guidelines are stored in private GitHub repositories. To access them you'll need:
  1. Install GitHub CLI: https://cli.github.com/
  2. Authenticate: gh auth login
  3. Request org access from your IT/security team if prompted

Proceeding without org config or guidelines for now.
You can re-run /hiring-setup after installing gh.
```
Set `github_available = false`. Continue to Step 4.

**Case B — gh installed but not authenticated:**
If output contains "not logged in" or "no credentials":
```
GitHub CLI is installed but not authenticated.

To authenticate:
  gh auth login

Then re-run /hiring-setup to pull org configs and guidelines.

Proceeding without org config or guidelines for now.
```
Set `github_available = false`. Continue to Step 4.

**Case C — gh authenticated but no org access:**
If `gh auth status` succeeds but subsequent org API calls return 404 or 403:
```
GitHub CLI is authenticated but you may not have access to the {org} organization yet.

To request org access:
  1. Visit: https://github.com/orgs/{org}/sso (if SSO is enabled)
  2. Or contact your IT/security team to be added to the GitHub org
  3. If your org uses SSO, you may need to authorize your token after joining:
     gh auth refresh -h github.com -s read:org

Proceeding without org config or guidelines for now.
```
Set `github_available = false`. Continue to Step 4.

**Case D — gh authenticated and org accessible:** Continue to Step 3a.

### Step 3a — Clone Org Config (Tier 2)

Resolve the org config repo:
- If `~/.hiring-assistant/org-config.yaml` exists and has `content.repo`: use that value
- Otherwise: try `{org}/hiring-assistant` as the default convention

Run:
```bash
gh api repos/{org}/{content_repo} --silent 2>&1
```

**If repo is accessible:**

1. Clone to a temp directory:
   ```bash
   temp_dir=$(mktemp -d)
   gh repo clone {org}/{content_repo} "$temp_dir" -- --depth 1
   ```

2. Copy org-config.yaml (if not already present locally, or if user confirms overwrite):
   ```
   if ~/.hiring-assistant/org-config.yaml exists:
     "Org config already exists locally. Overwrite with repo version? [Yes / No]"
   else:
     copy $temp_dir/org-config.yaml → ~/.hiring-assistant/org-config.yaml
   ```

3. Merge function configs and assets — for each function directory in `$temp_dir/functions/`:
   ```
   For each file in $temp_dir/functions/{function}/:
     target = ~/.hiring-assistant/functions/{function}/{relative_path}
     if target exists:
       skip (preserve local customizations)
     else:
       copy file to target
   ```

   Show:
   ```
   Org config synced from {org}/{content_repo}:
     ✓ org-config.yaml — {copied | already exists, skipped}
     ✓ functions/product-ops/config.yaml — {copied | already exists, skipped}
     ✓ functions/product-ops/templates/ — {N} files copied, {M} skipped (already exist)
     ✓ functions/product-ops/docs/ — {N} files copied
   ```

4. Clean up temp clone:
   ```bash
   rm -rf "$temp_dir"
   ```

**If repo is NOT accessible:**
```
Org config repo ({org}/{content_repo}) is not accessible.

This repo contains function configs, templates, and docs shared across your org.
To request access, contact your hiring lead or run:
  {access_request instructions from org-config or defaults}

You can still configure functions manually in Step 2b above.
```

### Step 3b — Acquire Guidelines (Tier 3)

Read `guidelines.repo_pattern` from `~/.hiring-assistant/org-config.yaml` (e.g., `hiring-guidelines-{function}`).
Read `guidelines.manual_fallback` (default: `true`).

For each function in `detected_functions`:

1. Resolve repo name: `{org}/{repo_pattern with {function} substituted}`
   - Map function ID to guidelines repo name using the pattern
   - If the function's config has a `guidelines_repo` field with a custom name, use that instead of the pattern
   - If the function's `config.yaml` has a `guidelines_repo` field, use that directly instead of the pattern

2. Check access:
   ```bash
   gh api repos/{org}/{resolved_guidelines_repo} --silent 2>&1
   ```

3. **If accessible:**
   - Check if `~/.hiring-assistant/guidelines/{function}/` already exists and is a git repo
     - If yes: `git -C ~/.hiring-assistant/guidelines/{function}/ pull`
     - If no: `git clone {org}/{resolved_guidelines_repo} ~/.hiring-assistant/guidelines/{function}/`
   - Show:
     ```
     ✓ {function} guidelines — cloned to ~/.hiring-assistant/guidelines/{function}/
     ```

4. **If NOT accessible:**
   - Show access request instructions:
     ```
     ✗ {function} guidelines — repo not accessible ({org}/{resolved_guidelines_repo})

     Guidelines contain the evaluation criteria for this function.
     They are stored in a restricted repo to protect interview integrity.
     ```

   - If org-config has `access_request.method` = "slack-bot":
     ```
     To request access, send this message to {bot_name}:
       "{access_request.prompts.add_to_group}" (filled with repo name)

     Or contact {guidelines.contact} via {guidelines.contact_method}.
     ```

   - If `guidelines.manual_fallback` is true:
     ```
     Alternatively, you can paste guidelines files manually.
     Would you like to:
       [1] Paste guidelines files now
       [2] Request access and come back later — run /hiring-admin sync after approval

     ```

     If user selects [1] (paste):
     ```
     Please provide the path to your guidelines file(s).
     These will be copied to ~/.hiring-assistant/guidelines/{function}/

     File or directory path:
     ```
     - Copy provided file(s) to `~/.hiring-assistant/guidelines/{function}/`
     - Show: `✓ Guidelines copied to ~/.hiring-assistant/guidelines/{function}/`

     If user selects [2]:
     ```
     No problem. You're set up for everything except guidelines-dependent evaluation.
     Once access is approved, run:
       /hiring-admin sync
     to pull the guidelines automatically.
     ```

Show final summary:
```
Guidelines status:
  ✓ product-ops — cloned
  ⏳ bco — access pending (run /hiring-admin sync after approval)
```

---

## Step 4 — Configure Transcript Source

Show available transcript adapters by reading all `.yaml` files in `adapters/transcript/`.

For each adapter, extract `id` and `description` fields.

Show:
```
How do you receive interview transcripts?

[1] BrightHire (copy from app)   — brighthire-clipboard
[2] Manual file (.md or .txt)    — manual-file
[3] Other (configure manually)
```

Ask user to select. Store as `transcript_adapter`.

---

## Step 5 — Configure ATS

Check if `GREENHOUSE_API_KEY` is set in the environment:

```bash
echo $GREENHOUSE_API_KEY
```

If set:
```
✓ Greenhouse API key found (GREENHOUSE_API_KEY)
```

If not set:
```
Greenhouse API key not found.

To enable Greenhouse integration:
  1. Obtain your API key from your People&Culture/HR team
  2. Add to your shell profile: export GREENHOUSE_API_KEY="your-key"
  3. Restart Claude Code

Proceeding without Greenhouse integration. You can re-run /hiring-setup after adding the key.
```

Store: `ats_configured = true/false`

---

## Step 5b — Calibration Examples (Optional)

Ask:
```
Do you have any existing assessment examples to use for calibration?

These are past assessments you trust — they help the evaluation agent anchor to your bar.

[Yes — I'll add them after setup] / [No / Skip]
```

If Yes:
```
After setup completes, place example assessments in:
  ~/.hiring-assistant/functions/{function}/examples/{level}/

Filename format: {yes|no|strong-yes|definitely-not}-example.md
```

---

## Step 5c — Data Privacy Settings

Interview data contains PII. This step sets your data handling policy — shown explicitly at setup so it is never a surprise.

**Load org defaults:**
- If `~/.hiring-assistant/org-config.yaml` exists and has a `data_privacy` section, pre-fill all values from it
- Otherwise use the regulatory baselines shown below

Show:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Data Privacy Settings
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Interview transcripts and assessments contain real candidate information.
These settings control how long that data is kept and how it is stored.

{If org-config loaded}:
  Your org has pre-configured privacy defaults below.
  You can confirm or override any setting.

── Transcript files ──

BrightHire retains the source recording. This controls the local copy only.

  [1] Delete after assessment is saved  ← recommended (source of truth stays in BrightHire)
  [2] Keep indefinitely
  [3] Never save to disk (process in-memory only)

  Org default: {org_config.data_privacy.transcript_persist | "no default set"}
  Your choice [1/2/3, or Enter to accept default]:

── Candidate data retention ──

Assessment files are kept in ~/.hiring-assistant/candidates/.
After this window, /hiring-admin clean will prompt for deletion.

  Regulatory baselines:
    Brazil (LGPD): 365 days — covers the 1-year CLT prescription period
                              for discrimination claims
    EU (GDPR):     180–365 days for rejected candidates
    US:            no statutory minimum; 1–2 years is common

  Org default: {org_config.data_privacy.retention_days | 365} days
  Retention window in days [Enter to accept default]:

── Cohort calibration storage ──

Cohort files track aggregated ratings to help calibrate future assessments.

  [1] Local — stored at ~/.hiring-assistant/cohort/ (private to your machine)
  [2] Disabled — skip cohort tracking

  Org default: {org_config.data_privacy.cohort_storage | "local"}
  Your choice [1/2, or Enter to accept default]:

── Cohort pseudonymization ──

When enabled, cohort rows use a candidate ID instead of the full name.
The assessment file still uses the real name — only the calibration aggregate is anonymized.

  Org default: {org_config.data_privacy.cohort_pseudonymize | false}
  Enable pseudonymization? [y/n, or Enter to accept default]:
```

Store: `privacy_transcript_persist`, `privacy_retention_days`, `privacy_cohort_storage`, `privacy_cohort_pseudonymize`.

---

## Step 6 — Check Optional Integrations

Check for Google Workspace MCP availability by attempting `calendar_listEvents` with `maxResults: 1`.

If available:
```
✓ Google Workspace MCP — calendar access confirmed
```

If unavailable:
```
⚠ Google Workspace MCP not available — calendar-based candidate detection will be disabled
  Manual entry will be used in /hiring-evaluate
```

Store: `calendar_enabled = true/false`

---

## Step 7 — Write Configuration

Construct the config object from all collected values and write to `~/.hiring-assistant/config.yaml`:

```yaml
# Hiring Assistant Configuration
# Generated by /hiring-setup on {date}

user:
  name: "{display_name}"
  email: "{email}"

functions: {detected_functions}   # e.g., [sales-engineering]

transcript:
  default_adapter: "{transcript_adapter}"   # e.g., brighthire-clipboard

ats:
  default_adapter: "greenhouse"
  configured: {ats_configured}

integrations:
  calendar: {calendar_enabled}

data_privacy:
  transcript_persist: "{privacy_transcript_persist}"   # always | delete_after_assessment | never
  retention_days: {privacy_retention_days}             # days to keep candidate files
  cohort_storage: "{privacy_cohort_storage}"           # local | disabled
  cohort_pseudonymize: {privacy_cohort_pseudonymize}   # true | false

setup_date: "{today}"
```

Show:
```
✅ Setup complete!

Configuration saved: ~/.hiring-assistant/config.yaml

Active functions: {functions list}
Transcript source: {adapter name}
Greenhouse: {✓ configured | ✗ not configured}
Calendar: {✓ enabled | ✗ not available}

Next steps:
- Run /hiring-evaluate to assess a candidate
- Run /hiring-help to see all available commands
- Add calibration examples to ~/.hiring-assistant/functions/{function}/examples/ (optional)
```

---

## Error Recovery

| Error | Recovery |
|-------|----------|
| `people_getMe` fails | Fall back to manual name/email entry |
| `gh` not installed | Proceed without guidelines; show install instructions |
| `gh auth status` fails (not logged in) | Proceed without guidelines; show `gh auth login` instructions |
| gh auth OK but org access denied | Proceed without guidelines; show SSO/org-join instructions |
| Calendar unavailable | Proceed without calendar; note manual entry will be required |
| Guidelines repo not found | Show access request instructions; offer to proceed without |
| Cannot write config file | Show config YAML to copy manually; check `~/.hiring-assistant/` dir permissions |

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
- "Product Ops", "Product Operations", "PO" → `product-ops`
- "Business Case", "BCO", "Business Consulting" → `bco`
- "PM", "Product Manager", "Product Management" → `product-manager`
- "Design", "UX", "Product Designer" → `design`

Show detected functions:
```
Detected from your calendar:
  ✓ product-ops (found {N} interviews in the past 30 days)

Additional functions available: (configure manually later with /hiring-admin)
```

Ask:
```
Is this correct? [Yes / No — let me choose manually]
```

If "No": show a text prompt to enter function IDs manually (comma-separated).

Store the confirmed list as `detected_functions`.

---

## Step 3 — Validate GitHub + Guidelines Access

### Step 3a — Check GitHub CLI

Run:
```bash
gh auth status 2>&1
```

**Case A — gh not installed:**
If command is not found:
```
GitHub CLI (gh) is not installed or not in your PATH.

Guidelines are stored in private GitHub repositories. To access them you'll need:
  1. Install GitHub CLI: https://cli.github.com/
  2. Authenticate: gh auth login
  3. Request org access from your IT/security team if prompted

Proceeding without guidelines access for now.
You can re-run /hiring-setup after installing gh.
```
Set `github_available = false`. Continue to Step 4.

**Case B — gh installed but not authenticated:**
If output contains "not logged in" or "no credentials":
```
GitHub CLI is installed but not authenticated.

To authenticate:
  gh auth login

Then re-run /hiring-setup to validate guidelines access.

Proceeding without guidelines access for now.
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

Proceeding without guidelines access for now.
```
Set `github_available = false`. Continue to Step 4.

**Case D — gh authenticated and org accessible:** Continue to Step 3b.

### Step 3b — Check Guidelines Repo Access

Read `~/.hiring-assistant/org-config.yaml` if present.

If org-config present and `guidelines.repo_pattern` is set:
- Pattern: `{guidelines.repo_pattern}` (e.g., `hiring-guidelines-{function}`)
- For each function in `detected_functions`:
  - Resolved repo: `{org}/{pattern with function substituted}`
  - Run: `gh api repos/{org}/{resolved-repo-name} --silent 2>&1`

Show access status:
```
Guidelines access:
  ✓ product-ops — repo accessible
  ✗ bco — repo not found or no access
```

For any function with missing access, show the request instructions from org-config:
```
To request access for {function}:
  {org_config.guidelines.message_template (filled in)}
  Contact: {org_config.guidelines.contact} via {org_config.guidelines.contact_method}
```

Ask:
```
Proceed without guidelines for inaccessible functions? [Yes / No]
```

If No: Stop. Re-run after requesting access.

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

functions: {detected_functions}   # e.g., [product-ops]

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

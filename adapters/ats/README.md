# ATS Adapters

Extensible registry for interacting with Applicant Tracking Systems. Each ATS platform = one YAML adapter + optional paired `.md` for output mapping. Drop a new `.yaml` to add support for a new platform — no core plugin changes required.

---

## How It Works

The ATS adapter is used at multiple points in the evaluation flow:

- **Candidate discovery** (Gate 1): `interview-router` uses `discovery.email_search_query` and `discovery.calendar_url_pattern` to find candidate ATS links
- **Materials download** (Gate 2): `materials-downloader` uses `attachments.endpoint` to fetch candidate PDFs
- **Scorecard read** (`/hiring-debrief prep-bar-raiser`): `debrief-synthesizer` uses `scorecard.read_endpoint` to fetch all submitted scorecards
- **ATS output** (Gate 7): `assessment-generator` uses `output_mapping.instructions` to format the ATS-ready section

---

## YAML Schema

```yaml
name: ""                      # Unique identifier (e.g., "greenhouse")
display_name: ""              # Display name (e.g., "Greenhouse")
api:
  base_url_pattern: ""        # URL template (e.g., "https://{domain}/v1")
  auth_method: ""             # "api_key" | "oauth" | "none"
  auth_env_var: ""            # Environment variable holding the key (e.g., "GREENHOUSE_API_KEY")
  auth_request_instructions: "" # How to request access from your org (shown in /hiring-setup)
url_patterns:
  candidate: ""               # Regex to extract application ID from ATS URL
discovery:
  email_search_query: ""      # Gmail search query for ATS interview reminders
  calendar_url_pattern: ""    # Regex to find ATS candidate links in calendar events
attachments:
  endpoint: ""                # API path template (e.g., "/applications/{application_id}/attachments")
  download_types: []          # Attachment types to download (e.g., ["resume", "cover_letter"])
  skip_types: []              # Attachment types to skip (e.g., ["other"])
  auth_header: ""             # Auth header format (e.g., "Basic {base64(api_key:)}")
scorecard:
  can_post: false             # Whether scorecard submission via API is supported
  endpoint: ""                # POST endpoint (if can_post: true)
  instructions: ""            # Path to .md with field mapping for POST
  read_endpoint: ""           # GET endpoint to read all submitted scorecards
  read_fields:                # Field paths in the API response to extract
    interviewer: ""
    stage: ""
    ratings: ""
    recommendation: ""
    submitted_at: ""
output_mapping:
  instructions: ""            # Path to .md describing the ATS scorecard format and field mapping
```

---

## Adding a New Adapter

1. Create `{ats-name}.yaml` following the schema
2. Create `{ats-name}-output.md` describing the ATS scorecard fields and how `assessment-generator` should populate them
3. Configure in `~/.hiring-assistant/org-config.yaml`:
   ```yaml
   integrations:
     ats:
       default_adapter: "{ats-name}"
   ```

---

## Active Adapters

| Adapter | Auth | Scorecard Read | Scorecard Post | Status |
|---------|------|---------------|----------------|--------|
| Greenhouse | API key | ✅ (`/v1/applications/{id}/scorecards`) | ❌ (not yet supported) | ✅ Active |

## Planned Adapters

| Adapter | Auth |
|---------|------|
| Lever | OAuth |
| Ashby | API key |
| Manual (no ATS) | None |

# hiring-assistant

A Claude Code plugin for AI-powered, structured interview evaluation. Produces evidence-based assessments with guidelines compliance tracking, cohort calibration, and ATS-ready output — across any job function your organization interviews for.

---

## What It Does

After each interview, run `/hiring-evaluate` and get a complete assessment in minutes:

- **Part 1 — Detailed analysis**: competency-by-competency scoring with transcript citations, gap analysis vs. level bar, red flag checklist, confidence scoring, and cohort calibration
- **Part 2 — ATS-ready**: Key Take-Aways, competency ratings with icons (👎/😐/👍/⭐), and Overall Recommendation (Definitely Not / No / Yes / Strong Yes) — ready to paste into Greenhouse

---

## Commands

| Command | Purpose |
|---------|---------|
| `/hiring-setup` | First-time onboarding (detects your function, configures integrations) |
| `/hiring-evaluate` | Post-interview assessment generation |
| `/hiring-prepare` | Pre-interview briefing from candidate materials |
| `/hiring-debrief` | Cross-stage debrief synthesis for bar raisers |
| `/hiring-admin` | Function setup, validation, sync, backtest |
| `/hiring-help` | Show commands, config status, upcoming interviews |

---

## Architecture

```
hiring-assistant/                    PUBLIC — engine only (this repo)
├── commands/                        Slash command orchestration logic
├── agents/                          Specialized subagents (router, evaluator, etc.)
├── adapters/
│   ├── material/                    Extensible registry: drop a YAML = new file type
│   ├── transcript/                  Extensible registry: drop a YAML = new transcript source
│   └── ats/                         Extensible registry: drop a YAML = new ATS platform
├── core/schemas/                    What guidelines, configs, and outputs must contain
└── templates/                       Templates for adding new functions

~/.hiring-assistant/                 PRIVATE — your org's content (never in this repo)
├── config.yaml                      Your user config (created by /hiring-setup)
├── org-config.yaml                  Your org's configuration
├── functions/{function}/            Competency frameworks, templates, function configs
├── guidelines/{function}/           Cloned private guidelines repos
├── candidates/{name}/               Candidate runtime files (transcripts, assessments)
└── cohort/{function}/{type}/        Calibration tracking across candidates
```

**Public/private boundary:** This repo contains zero evaluation criteria, competency definitions, or org-specific content. All of that lives under `~/.hiring-assistant/` and in your organization's private repos.

---

## Requirements

- [Claude Code](https://claude.ai/claude-code) installed
- GitHub CLI (`gh`) configured with access to your org
- Access to your org's guidelines repos (private, per-function)
- *(Optional)* Greenhouse API key — enables automatic scorecard fetching and debrief synthesis
- *(Optional)* Google Workspace MCP — enables calendar-based candidate detection and transcript fetching
- *(Optional)* BrightHire — provides interview transcripts (clipboard paste supported without MCP)

---

## Installation

### Via Local Marketplace

If your org has registered this plugin in a local marketplace:

```
/plugin install hiring-assistant
```

### Manual Installation

1. Clone or download this repo
2. Register in your Claude Code settings (see your org's setup guide)
3. Run `/hiring-setup` to configure

---

## Quick Start

```
/hiring-setup
```

Follow the 7-step wizard. It will:
1. Detect your identity and active functions from Google Calendar
2. Validate GitHub and guidelines access
3. Configure your transcript source (BrightHire, Google Meet, or manual)
4. Configure your ATS (Greenhouse or other)
5. Optionally copy existing assessments as calibration examples
6. Check available material reader integrations
7. Write `~/.hiring-assistant/config.yaml`

Then, after your next interview:

```
/hiring-evaluate
```

---

## Adding Your Org

The hiring assistant is a generic engine. All org-specific content (guidelines, competency definitions, evaluation criteria) lives under `~/.hiring-assistant/` and in your organization's private repos — never in this public repo.

To adopt the engine for your org:

**1. Copy the org template**

```bash
cp org-config-template.yaml ~/.hiring-assistant/org-config.yaml
```

Edit `org-config.yaml` to set your org name, guidelines repo base URL, and ATS configuration.

**2. Create your first function directory**

```bash
mkdir -p ~/.hiring-assistant/functions/{your-function}/templates
```

Copy the scaffold config and edit it:

```bash
cp templates/function/config.yaml ~/.hiring-assistant/functions/{your-function}/config.yaml
```

Define your interview types, levels, and competencies in that config.

**3. Clone your private guidelines repo**

```bash
git clone {your-private-guidelines-repo} ~/.hiring-assistant/guidelines/{your-function}/
```

The guidelines repo must follow the schema in `core/schemas/guidelines-schema.md`.

**4. Run `/hiring-setup`**

```
/hiring-setup
```

The setup wizard validates your config, checks guidelines access, and writes your user `config.yaml`.

---

## Adding New Functions

Each new job function requires:
1. A `config.yaml` defining interview types, levels, and competencies (template: `templates/function/config.yaml`)
2. Private guidelines repos cloned to `~/.hiring-assistant/guidelines/{function}/`
3. Function-specific templates under `~/.hiring-assistant/functions/{function}/templates/`

See `core/schemas/function-config-schema.md` for the full schema, and your org's `org-config.yaml` for the guidelines repo pattern.

## Adding New Material Readers / Adapters

Drop a `.yaml` file (and optional paired `.md`) into the relevant registry directory. No core plugin changes required.

- `adapters/material/` — how to extract content from a file or URL
- `adapters/transcript/` — how to acquire interview transcripts
- `adapters/ats/` — how to interact with your hiring platform

See each directory's `README.md` for the schema.

---

## Data Privacy

Interview data contains real candidate information (names, transcripts, competency ratings). The plugin is designed to handle this responsibly.

### What gets stored locally

| Data | Path | Lifetime |
|------|------|---------|
| Interview transcript | `~/.hiring-assistant/candidates/{name}/transcript.md` | Configurable (see below) |
| Assessment (detailed + ATS) | `~/.hiring-assistant/candidates/{name}/assessment-{type}.md` | Configurable retention window |
| Cohort calibration rows | `~/.hiring-assistant/cohort/{function}/{type}/` | Long-lived; pseudonymized by default |
| Interviewer notes | `~/.hiring-assistant/candidates/{name}/notes.md` | Configurable retention window |

### Configurable privacy settings

Set in `org-config.yaml` (org-wide policy) or `config.yaml` (user override):

```yaml
data_privacy:
  transcript_persist: "delete_after_assessment"  # always | delete_after_assessment | never
  retention_days: 365                             # 0 = disable cleanup prompts
  cohort_storage: "local"                         # local | confluence | s3 | disabled
  cohort_pseudonymize: true                       # use candidate ID instead of name in cohort rows
```

**`transcript_persist`** — The transcript is only needed to generate the assessment. `delete_after_assessment` removes it once the assessment file is saved; the source recording (e.g. BrightHire) remains intact.

**`retention_days`** — After this window, `/hiring-admin clean` flags candidate files for deletion. Regulatory baselines: Brazil (LGPD) 365 days, EU (GDPR) 180–365 days for rejected candidates.

**`cohort_pseudonymize`** — Cohort rows store a deterministic candidate ID (hash of name + date) instead of the full name. The assessment file retains the real name; only the long-lived aggregate is anonymized.

### Directory permissions

`/hiring-setup` sets `chmod 700` on `~/.hiring-assistant/` and `chmod 600` on `config.yaml`, ensuring only your user account can read any of the data. Enable full-disk encryption (e.g. macOS FileVault) for at-rest protection.

---

## Repository Structure

```
hiring-assistant/
├── plugin.json
├── README.md
├── skills/
│   ├── evaluate-candidate/SKILL.md
│   ├── setup-wizard/SKILL.md
│   ├── hiring-help/SKILL.md
│   ├── hiring_prepare/SKILL.md
│   ├── hiring_debrief/SKILL.md
│   └── hiring_admin/SKILL.md
├── commands/
│   ├── evaluate.md
│   ├── setup.md
│   ├── help.md
│   ├── prepare.md
│   ├── debrief.md
│   └── admin.md
├── agents/
│   ├── interview-router.md
│   ├── guideline-loader.md
│   ├── transcript-fetcher.md
│   ├── assessment-generator.md
│   ├── cohort-updater.md
│   ├── material-reader.md
│   ├── prepare-briefing.md
│   └── debrief-synthesizer.md
├── adapters/
│   ├── material/            (pdf.yaml + pdf.md)
│   ├── transcript/          (brighthire-clipboard.yaml, manual-file.yaml)
│   └── ats/                 (greenhouse.yaml, greenhouse-output.md)
├── core/
│   ├── workflow.md
│   └── schemas/
├── templates/
│   └── function/
└── org-config-template.yaml
```

---

## License

MIT — engine only. Your org's evaluation criteria, guidelines, and templates are private and not included.

# Guidelines Structure

This document defines the expected structure for guidelines repositories. Each job function has its own private guidelines repo, cloned to `~/.hiring-assistant/guidelines/{function}/` during `/hiring-setup`.

---

## Required Directory Structure

```
{guidelines-repo}/
├── behavioral/
│   └── {Function} - Behaviour Evaluation Guide.md    # Required for behavioral interviews
└── business-case/
    └── {case-name}/                                   # One subfolder per case variant
        ├── evaluation-guide.md                        # Required: how to evaluate the case
        └── candidate-materials/                       # Optional: files given to candidates
            └── {Case Material}.pdf
```

---

## What Guidelines Must Contain

Every guidelines document must satisfy the schema defined in `core/schemas/guidelines-schema.md`. At minimum:

### For Behavioral Interviews

- **Competencies section**: List of competencies being evaluated, with definitions and evaluation criteria
- **Level bars**: Expectations per IC level (IC4, IC5, IC6, IC7, and manager equivalents)
- **Signals**: Green flags (positive indicators) and red flags (concerns) — at least 5 each
- **Interview questions** mapped to competencies (Q1-Q20 or similar)

### For Business Case Interviews

- **Competencies section**: Skills and knowledge being evaluated
- **Level bars**: Expectations per IC level
- **Signals**: Green and red flags
- **Must-identify insights**: Key findings the candidate should discover, with importance ratings (critical / important / nice-to-have)
- **Probing questions**: Questions to ask if the candidate misses key insights

---

## Setting Up Guidelines Access

Guidelines repos are private and access is per-function. When `/hiring-setup` detects that a guidelines repo is inaccessible, it guides the user through the access request flow configured in `org-config.yaml` (typically a Slack message to the hiring lead).

To manually request access:
```
Contact your org's hiring lead (see org-config.yaml → guidelines.contact)
```

To clone once access is granted:
```bash
git clone {org-guidelines-repo-url} ~/.hiring-assistant/guidelines/{function}
```

To update to the latest guidelines:
```bash
cd ~/.hiring-assistant/guidelines/{function} && git pull
```

Or use the sync command:
```
/hiring-admin sync
```

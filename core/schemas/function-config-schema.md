# Function Config Schema

Defines the required structure for `config.yaml` files under `~/.hiring-assistant/functions/{function}/`.

This schema is enforced by `guideline-loader` before any evaluation, and by `/hiring-admin validate`.

---

## Full Schema

```yaml
# ─────────────────────────────────────────
# Function identity
# ─────────────────────────────────────────
function: ""                      # Unique identifier, lowercase-hyphenated (e.g., "product-ops")
name: ""                          # Display name (e.g., "Product Operations")
status: "active"                  # "active" | "draft" — only active functions can be evaluated
guidelines_repo: ""               # Org's private guidelines repo name (e.g., "hiring-guidelines-{function}")

# ─────────────────────────────────────────
# Interview types
# ─────────────────────────────────────────
interview_types:
  - id: ""                        # e.g., "behavioral"
    name: ""                      # e.g., "Behavioral Interview"
    levels: []                    # Supported levels: [IC4, IC5, IC6, IC7] (M2=IC6, M3=IC7)
    async: false                  # true = no live recording; transcript is optional, not required
    required_inputs:
      - type: ""                  # "transcript" | "presentation" | "resume" | "notes"
        formats: []               # Accepted extensions: ["md", "txt", "pdf", "pptx"]
    optional_inputs:
      - type: "notes"
        formats: ["md", "txt"]
    competencies:                 # List of competency IDs (must match guidelines document)
      - id: ""
        name: ""
    guidelines_path: ""           # Relative path within guidelines repo (e.g., "behavioral/")

  - id: "business-case"
    name: ""
    levels: []
    async: false
    cases:                        # One entry per case variant
      - id: ""                    # e.g., "authorizer"
        name: ""                  # e.g., "Credit Card Authorization"
        active: true              # Only active cases are offered in /hiring-evaluate
        guidelines_path: ""       # Relative path in guidelines repo (e.g., "business-case/authorizer/")
        candidate_materials: []   # Filenames of materials given to the candidate
    required_inputs: []
    optional_inputs: []
    competencies: []
    guidelines_path: ""           # Default path if no case-specific path

# ─────────────────────────────────────────
# ATS integration
# ─────────────────────────────────────────
greenhouse:
  scorecard_prefix: ""            # Prefix used when posting to ATS (e.g., function short name)
```

---

## Validation Rules

`guideline-loader` checks the following before proceeding:

- `function` and `name` are non-empty strings
- `status` is `"active"` (hard stop if `"draft"`)
- At least one entry in `interview_types`
- For each interview type:
  - `id`, `name`, `levels` are present and non-empty
  - `required_inputs` has at least one entry
  - `competencies` has at least one entry
  - `guidelines_path` is non-empty
- For business-case types: at least one case with `active: true`

---

## Level Reference

| Input Level | Evaluates As | Career Track |
|-------------|--------------|-------------|
| IC4 | IC4 | Individual Contributor |
| IC5 | IC5 | Individual Contributor |
| IC6 | IC6 | Individual Contributor |
| M2 | IC6 | People Manager |
| IC7 | IC7 | Individual Contributor |
| M3 | IC7 | People Manager |

Manager-track levels (M2, M3) use the same evaluation bar as their IC equivalents.

---

## Example: Product Operations

```yaml
function: "product-ops"
name: "Product Operations"
status: "active"
guidelines_repo: "hiring-guidelines-product-ops"

interview_types:
  - id: "behavioral"
    name: "Behavioral Interview"
    levels: [IC4, IC5, IC6, M2, IC7, M3]
    async: false
    required_inputs:
      - type: transcript
        formats: [md, txt]
    optional_inputs:
      - type: notes
        formats: [md, txt]
    competencies:
      - id: stakeholder-management
        name: Stakeholder Management
      - id: continuous-improvement
        name: Continuous Improvement
      - id: achievements
        name: Achievements
      - id: communication
        name: Communication
      - id: english-level
        name: English Level
      - id: fast-learner
        name: Fast Learner
      - id: ownership
        name: Ownership
      - id: smart-problem-solver
        name: Smart Problem Solver
      - id: result-oriented
        name: Result Oriented
    guidelines_path: "behavioral/"

  - id: "business-case"
    name: "Business Case"
    levels: [IC4, IC5, IC6, M2, IC7, M3]
    async: true
    cases:
      - id: "authorizer"
        name: "Credit Card Authorization"
        active: true
        guidelines_path: "business-case/authorizer/"
        candidate_materials: []
      - id: "bills-payments"
        name: "Bills & Payments"
        active: true
        guidelines_path: "business-case/bills-payments/"
        candidate_materials: []
    required_inputs:
      - type: presentation
        formats: [pdf, pptx, key]
    optional_inputs:
      - type: notes
        formats: [md, txt]
      - type: transcript
        formats: [md, txt]
    competencies:
      - id: problem-identification
        name: Problem Identification
      - id: analytical-thinking
        name: Analytical Thinking
      - id: solution-design
        name: Solution Design
      - id: business-sense
        name: Business Sense
      - id: communication
        name: Communication
      - id: stakeholder-management
        name: Stakeholder Management
    guidelines_path: "business-case/"

greenhouse:
  scorecard_prefix: "product-ops"
```

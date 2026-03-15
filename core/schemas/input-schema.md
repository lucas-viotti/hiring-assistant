# Input Schema

This document defines what **inputs** are required for each interview type.

---

## Purpose

Different interview types require different input materials. This schema ensures all necessary inputs are present before evaluation begins.

---

## Input Types

### 1. Transcript

**Description:** Interview transcript from BrightHire

| Field | Value |
|-------|-------|
| Type | `transcript` |
| Formats | `.md`, `.txt` |
| Source | BrightHire via Greenhouse |
| Required | Always |

**Naming convention:** `[Level] - [Name] - Transcript.md`

### 2. Presentation

**Description:** Candidate's case presentation

| Field | Value |
|-------|-------|
| Type | `presentation` |
| Formats | `.pdf`, `.pptx` |
| Source | Candidate submission |
| Required | Business case interviews only |

**Naming convention:** `[Level] - [Name] - Case Presentation.pdf`

### 3. Notes

**Description:** Interviewer notes from BrightHire

| Field | Value |
|-------|-------|
| Type | `notes` |
| Formats | `.md` |
| Source | BrightHire AI notes |
| Required | Optional |

**Naming convention:** `[Level] - [Name] - Notes.md`

---

## Requirements by Interview Type

### Behavioral Interview

| Input | Required | Format |
|-------|----------|--------|
| Transcript | Yes | .md, .txt |
| Notes | Optional | .md |

### Business Case Interview

| Input | Required | Format |
|-------|----------|--------|
| Transcript | Yes | .md, .txt |
| Presentation | Yes | .pdf, .pptx |
| Notes | Optional | .md |

### Portfolio Review (Design)

| Input | Required | Format |
|-------|----------|--------|
| Transcript | Yes | .md, .txt |
| Portfolio | Yes | .pdf, link |
| Notes | Optional | .md |

---

## Filename Convention

**Standard format:** `[Level] - [Full Name] - [Type] - [Document].ext`

| Component | Description | Examples |
|-----------|-------------|----------|
| Level | IC level | IC4, IC5, IC6, IC7 |
| Full Name | Candidate name | Maria Santos |
| Type | Interview type (optional) | Behavioral, Business Case |
| Document | Document type | Transcript, Case Presentation, Assessment |

**Examples:**
```
IC6 - Maria Santos - Transcript.md
IC6 - Maria Santos - Case Presentation.pdf
IC6 - Maria Santos - Business Case - Assessment.md
IC5 - João Silva - Behavioral - Assessment.md
```

---

## Level Extraction

The AI should extract candidate level from:

1. **Filename** — Look for IC4, IC5, IC6, IC7 pattern
2. **User prompt** — "Evaluate for IC6"
3. **Folder path** — `candidates/IC6/`

If level cannot be determined, ASK the user.

---

## Validation Rules

### Pre-Flight Check

Before starting evaluation:

```
□ Candidate level confirmed (IC4/IC5/IC6/M2/IC7/M3)
□ Required inputs present for interview type
□ File formats match expected
□ Guidelines accessible
□ Output folder exists
```

### If Inputs Missing

| Missing Input | Action |
|---------------|--------|
| Transcript | Cannot proceed — ask user to provide |
| Presentation (case) | Cannot proceed — ask user to provide |
| Notes | Can proceed — note limited context |
| Level | Ask user to specify |

---

## Input Quality Checks

### Transcript Quality

- [ ] Contains interview questions and answers
- [ ] Candidate and interviewer clearly identified
- [ ] Timestamps or structure present
- [ ] Not obviously truncated

### Presentation Quality

- [ ] All slides readable
- [ ] Text extractable (not image-only)
- [ ] Complete (not missing sections)
- [ ] Reasonable length for case type

---

## File Location

Input files are saved under the candidate's runtime directory:

```
~/.hiring-assistant/candidates/{candidate-name}/
├── transcript.md
├── materials/
│   ├── {Name} - Case Presentation.pdf
│   └── {Name} - Case Presentation - Extracted.md
├── notes.md                     (optional)
├── assessment-behavioral.md
└── assessment-business-case.md
```

> ⚠️ Candidate data is **never committed to any repository**. It lives only at `~/.hiring-assistant/`.


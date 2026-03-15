# Guidelines Schema

This document defines what every **guideline document** MUST contain to be valid.

---

## Purpose

Guidelines are the **source of truth** for evaluation criteria. They live in the private `guidelines/` submodule and define:
- What competencies to evaluate
- What the bar is for each level
- What signals to watch for

---

## Required Sections

Every guideline document MUST contain these sections:

### 1. Competencies

**Purpose:** Define what skills are being evaluated

**Required fields:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (e.g., "stakeholder-management") |
| `name` | string | Display name (e.g., "Stakeholder Management") |
| `definition` | string | What this competency measures |
| `evaluation_criteria` | list | Specific things to look for |

**Example:**
```markdown
## Competencies

### Stakeholder Management
- **Definition:** Ability to identify, engage, and align stakeholders
- **Evaluation Criteria:**
  - Stakeholder identification
  - Engagement & alignment
  - Relationship building
  - Cross-functional leadership
```

### 2. Level Bars

**Purpose:** Define expectations for each IC level

**Required fields:**
| Field | Type | Description |
|-------|------|-------------|
| `level` | enum | IC4, IC5, IC6, IC7, M2, M3 |
| `expectations` | list | What candidates at this level should demonstrate |
| `acceptable_gaps` | list | Minor gaps that are OK at this level |

**Example:**
```markdown
## Level Bars

### IC6 Expectations
| Competency | Expectation |
|------------|-------------|
| Stakeholder Management | Influences across functions without authority |
| Achievements | 3-5 projects with significant business impact |

### IC6 Acceptable Gaps
- Minor open points/mistakes
- Can answer about missing insights when probed
```

### 3. Signals

**Purpose:** Define green flags (positive) and red flags (concerns)

**Required fields:**
| Field | Type | Description |
|-------|------|-------------|
| `green_flags` | list | Positive signals to watch for |
| `red_flags` | list | Concerning signals to watch for |

**Example:**
```markdown
## Signals

### Green Flags
- Quantified results with clear individual role
- Influence without authority demonstrated
- Self-awareness about failures and learning

### Red Flags
- No quantified results
- Vague or generic examples
- Cannot articulate specific contribution
```

---

## Optional Sections

These sections are required for specific interview types:

### 4. Interview Questions (Behavioral Only)

**Purpose:** Map questions to competencies

**Required fields:**
| Field | Type | Description |
|-------|------|-------------|
| `competency` | string | Which competency this question probes |
| `question` | string | The question to ask |
| `follow_ups` | list | Probing questions if needed |

### 5. Must-Identify Insights (Case-Based Only)

**Purpose:** Define key insights candidates should find

**Required fields:**
| Field | Type | Description |
|-------|------|-------------|
| `insight` | string | Description of the insight |
| `evidence` | string | Where in the case this appears |
| `importance` | enum | critical, important, nice-to-have |

**Example:**
```markdown
## Must-Identify Insights

| Insight | Evidence | Importance |
|---------|----------|------------|
| NuConta is optimal channel | Cost data, stability data | Critical |
| Bill slip efficiency declining | Processed/generated ratio | Critical |
| Bank A vs B cost trade-off | Supplier cost breakdown | Important |
```

### 6. Probing Questions (Case-Based Only)

**Purpose:** Questions to ask if candidate misses insights

**Example:**
```markdown
## Probing Questions

If candidate doesn't identify supplier cost trade-off:
- "Looking at the cost data, what differences do you see between suppliers?"
- "For what types of customers would each supplier be better?"
```

---

## Validation Checklist

Before using a guideline document, verify:

- [ ] **Competencies defined** with id, name, definition
- [ ] **Level bars present** for all supported levels
- [ ] **Green flags listed** (at least 5)
- [ ] **Red flags listed** (at least 5)
- [ ] **For behavioral:** Interview questions mapped to competencies
- [ ] **For case-based:** Must-identify insights listed with importance

---

## File Location

Guidelines are stored in the private runtime directory (cloned from your org's private repo):

```
~/.hiring-assistant/guidelines/
├── [function]/
│   ├── behavioral/
│   │   └── [Guideline Document].md
│   └── business-case/
│       └── [case-name]/
│           ├── evaluation-guide.md
│           └── candidate-materials...
```

The guidelines path for each interview type is defined in `~/.hiring-assistant/functions/{function}/config.yaml` → `interview_types[*].guidelines_path`.

---

## Schema Validation

When loading guidelines, the AI should verify:

1. Document exists at configured path
2. All required sections present
3. Level bars match interview type's supported levels
4. Competencies have all required fields


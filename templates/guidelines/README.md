# Guidelines Structure

This document defines the expected structure for guidelines repositories. Each job function has its own private guidelines repo, cloned to `~/.hiring-assistant/guidelines/{function}/` during `/hiring-setup`.

---

## Required Directory Structure

```
{guidelines-repo}/
├── behavioral/
│   └── {Function} - Behaviour Evaluation Guide.md    # Required for behavioral interviews
├── business-case/
│   └── {case-name}/                                   # One subfolder per case variant
│       ├── evaluation-guide.md                        # Required: how to evaluate the case
│       └── candidate-materials/                       # Optional: files given to candidates
│           └── {Case Material}.pdf
├── technical/                                         # Optional: coding / technical interviews
│   └── {Function} - Technical Evaluation Guide.md
├── system-design/                                     # Optional: architecture / design interviews
│   └── {Function} - System Design Evaluation Guide.md
└── portfolio/                                         # Optional: portfolio / work sample reviews
    └── {Function} - Portfolio Evaluation Guide.md
```

Not every function uses every interview type. Include only the directories that match your function's `config.yaml` interview types.

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

### For Technical / Coding Interviews

- **Competencies section**: Technical skills being evaluated (e.g., problem decomposition, code quality, algorithm choice, testing approach)
- **Level bars**: Expectations per level (what distinguishes an IC4 solution from an IC7)
- **Signals**: Green flags (clean abstractions, considers edge cases, tests first) and red flags (brute force without optimization, no error handling)
- **Evaluation rubric**: How to score code quality, correctness, efficiency, and communication

### For System Design Interviews

- **Competencies section**: Design skills being evaluated (e.g., scalability reasoning, trade-off analysis, component decomposition, data modeling)
- **Level bars**: Expectations per level (IC5 may scope a single service; IC7 should reason about cross-system interactions)
- **Signals**: Green flags (drives clarifying questions, quantifies constraints, considers failure modes) and red flags (jumps to solution without requirements, ignores non-functional requirements)
- **Evaluation rubric**: How to score breadth vs. depth, trade-off articulation, and communication

### For Portfolio / Work Sample Reviews

- **Competencies section**: What to evaluate in the candidate's past work (e.g., impact framing, technical depth, design quality, stakeholder management)
- **Level bars**: Expectations per level for quality and scope of work presented
- **Signals**: Green flags (quantified impact, clear ownership narrative, lessons from failures) and red flags (vague contributions, no measurable outcomes, only team-level results)
- **Evaluation rubric**: How to score impact, complexity, and storytelling

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

# Evaluation Workflow

The universal evaluation flow for `/hiring-evaluate`. All functions and interview types follow this gate sequence.

---

## Gate Sequence

```
/hiring-evaluate [--auto | --confirm]

GATE 1 ─ CANDIDATE DISCOVERY (interview-router)
  Search Google Calendar + Gmail for recent past interviews
  Extract: candidate name, level, function, interview type, ATS link
  Present match(es) → user confirms, or manual entry fallback
  ↓
GATE 2 ─ MATERIALS ACQUISITION (materials-orchestrator) [business case only]
  Detect material types from ATS attachments and/or user-provided files
  Dispatch to matching material readers in parallel
  Fallback: ask user to provide files manually
  ↓
GATE 3 ─ CASE VARIANT DETECTION [business case only, multiple variants exist]
  Analyze candidate materials against known case variants from function config
  Match content signatures (keywords, data patterns) to case ID
  Confirm with user (or auto-detect and skip if --auto and unambiguous)
  ↓
GATE 4 ─ GUIDELINES (guideline-loader)
  Load guidelines for confirmed function + interview type + case variant
  Validate against guidelines-schema.md
  Hard stop if guidelines missing or invalid → guide user to /hiring-setup
  ↓
GATE 4b ─ RE-EVALUATION DETECTION
  Check if extracted/enriched files already exist for this candidate + type
  If exists and < 24h old: offer reuse vs re-extract
  --auto: reuse if < 24h, re-extract otherwise
  ↓
GATE 5 ─ TRANSCRIPT (transcript-fetcher)     ══ PARALLEL ══     GATE 6 ─ DEEP EXTRACTION
  Check if interview type has async: true                         [business case only]
  Load configured transcript adapter                              Further extraction of materials
  Execute acquisition method (clipboard/file/mcp)                PDF → markdown + slide images
  Run enrichment pipeline steps                                   via pdf-extractor agent
  Save to ~/.hiring-assistant/candidates/{name}/transcript.md
  If unavailable: proceed with notes only, warn on confidence
  ↓
GATE 5b ─ SUPPLEMENTARY NOTES (optional)
  Load interviewer notes if present in candidates/{name}/notes.md
  Pass as supplementary input alongside transcript
  ↓
GATE 7 ─ EVALUATION (assessment-generator)
  Load assessment-detailed-{type}.md  ← structure template
  Load output-spec-{type}.md          ← quality rules / anti-patterns
  Load calibration examples if available
  Use extended thinking to reason through each competency with evidence
  Generate Part 1: Detailed analysis (confidence, gap table, red flags, cohort calibration)
  Generate Part 2: ATS-ready (key take-aways, competency ratings, recommendation)
  Self-check against output-spec rules before presenting
  Present to user → confirm or revise (skipped if --auto)
  ↓
GATE 8 ─ FINALIZE (cohort-updater)
  Append summary row to ~/.hiring-assistant/cohort/{function}/{type}/all-assessments.md
  Update level-specific calibration file
  Print success message with ATS link
```

---

## Flags

| Flag | Behavior |
|------|----------|
| *(none)* | Default: confirm at each gate before proceeding |
| `--auto` | Skip all confirmations; auto-pick most recent candidate, auto-detect case, auto-reuse files |
| `--confirm` | Pause at every gate even for trivially-clear decisions (useful for first use) |

---

## Notes-Only Path

When a transcript is unavailable (interview type has `async: true`, or user skips):
- Evaluation proceeds using interviewer notes + candidate materials only
- Confidence auto-flagged as **Low** (no transcript) or **Medium** (short transcript < 30 min)
- Assessment output includes a `⚠️ Limited Evidence` banner

---

## Re-Evaluation Path (Gate 4b)

If assessment files already exist for the same candidate + interview type:
- Offer to update only specific sections vs. full re-run
- Extracted files (transcript.md, slide markdowns) are reused by default if < 24 hours old
- This prevents redundant PDF extractions and transcript enrichment passes

---

## Agent Dispatch Map

| Gate | Agent |
|------|-------|
| 1 | `interview-router` |
| 2 | `materials-orchestrator` → `materials-downloader` |
| 3 | (inline in `evaluate.md`) |
| 4 | `guideline-loader` |
| 5 | `transcript-fetcher` |
| 6 | `pdf-extractor` (via `materials-orchestrator`) |
| 7 | `assessment-generator` |
| 8 | `cohort-updater` |

# Manual File — Transcript Parse Instructions

<role>
You are the transcript-fetcher agent handling a manually-provided transcript file. Read the file and normalize it to the standard transcript format.
</role>

## Input

```
<file_path>{user-provided path}</file_path>
<candidate_name>{name from routing result}</candidate_name>
```

## Parse Process

### Step 1: Read the file

Use the Read tool to load the file at the provided path.

If the file does not exist or cannot be read:
- Ask the user to verify the path
- Offer to proceed with notes-only evaluation

### Step 2: Detect and normalize format

Same normalization rules as `brighthire-clipboard.md` — convert to speaker-labeled markdown with timestamps or sequential numbering.

### Step 3: Apply enrichment pipeline

Run enabled enrichment steps from `~/.hiring-assistant/config.yaml` → `transcript_enrichment.enabled_steps`.

### Step 4: Save

Copy/save normalized content to:
`~/.hiring-assistant/candidates/{candidate-name}/transcript.md`

Add provenance header:
```markdown
---
source: manual-file
original_path: {file_path}
candidate: {candidate_name}
date: {today}
enrichment: {list of applied enrichment steps}
---
```

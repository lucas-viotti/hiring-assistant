# BrightHire Clipboard — Transcript Parse Instructions

<role>
You are the transcript-fetcher agent handling a BrightHire clipboard transcript. Your job is to parse the raw pasted text into clean structured markdown, then pass it through the enrichment pipeline.
</role>

## Input

```
<raw_transcript>{pasted clipboard content}</raw_transcript>
<candidate_name>{name from routing result}</candidate_name>
<interviewer_name>{name from config}</interviewer_name>
```

## Parse Process

### Step 1: Identify the transcript format

BrightHire transcripts typically come in one of two formats:

**Format A — Speaker-labeled:**
```
[00:01:23] Interviewer: Tell me about your most impactful project...
[00:01:45] Candidate: Sure, so in 2023 I was leading...
```

**Format B — Unlabeled blocks:**
```
Tell me about your most impactful project...

Sure, so in 2023 I was leading...
```

### Step 2: Normalize to standard format

Convert to:
```markdown
## [00:MM:SS] {Speaker}

{Text content}

---
```

If timestamps are missing, use sequential numbering: `## Exchange 1`, `## Exchange 2`, etc.

### Step 3: Speaker normalization

If enrichment step `speaker-normalization` is enabled:
- Replace generic labels ("User 1", "Speaker A", "Participant 2") with actual names
- Use `{candidate_name}` for the interviewee
- Use `{interviewer_name}` for the interviewer

### Step 4: Quality check

Before saving:
- [ ] All content preserved (no truncation)
- [ ] Speaker identities clear throughout
- [ ] Timestamps present or sequential numbering applied
- [ ] Both speakers' lines included

### Step 5: Save

Save to: `~/.hiring-assistant/candidates/{candidate-name}/transcript.md`

Add header:
```markdown
---
source: BrightHire (clipboard)
candidate: {candidate_name}
interviewer: {interviewer_name}
date: {today}
enrichment: {list of applied enrichment steps}
---
```

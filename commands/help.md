# /hiring-help

Shows your current configuration, active functions, upcoming interviews, and command reference.

**Usage:** `/hiring-help`

---

## Pre-flight

If `~/.hiring-assistant/config.yaml` is missing:
```
Hiring Assistant is not configured yet.

Run /hiring-setup to get started.
```
Stop.

---

## Step 1 — Read Configuration

Read `~/.hiring-assistant/config.yaml`.

Also read `~/.hiring-assistant/org-config.yaml` if present.

---

## Step 2 — Show Status

Display:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Hiring Assistant  v0.1.0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Configured for: {user.name} ({user.email})
Setup date: {setup_date}

Active functions:
  • {function_name} — {interview_types} ({status})

Transcript source: {transcript.default_adapter}
ATS: {ats status — "✓ Greenhouse configured" | "✗ Not configured"}
Calendar: {"✓ Enabled" | "✗ Not available"}
```

---

## Step 3 — Upcoming Interviews

If `integrations.calendar = true`:

Use `calendar_listEvents` to fetch the next 7 days:

```
timeMin: {today}
timeMax: {7 days from today}
q: "interview"
maxResults: 20
```

Filter events matching interview patterns (same keywords as `/hiring-setup` Step 2).

Show:

```
Upcoming interviews (next 7 days):
  {day, date}  {time}  {candidate name or event title}  [{function}]
  ...

  No upcoming interviews found.  (if empty)
```

If calendar unavailable: skip this section silently.

---

## Step 4 — Cohort Summary

For each active function, check if cohort files exist at:
`~/.hiring-assistant/cohort/{function}/behavioral/all-assessments.md`

If file exists, count rows in the Assessment Log table (lines starting with `|` after the header).

Show:

```
Cohort data:
  product-ops / behavioral — {N} assessments logged
  (no data yet)             — if file missing
```

---

## Step 5 — Command Reference

Show:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Commands
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/hiring-evaluate [--auto | --confirm]
  Run a structured interview evaluation.
  --auto: skip all confirmations
  --confirm: pause at every gate (recommended for first use)

/hiring-setup
  Configure or reconfigure the assistant.
  Re-run to add functions, change transcript source, or update integrations.

/hiring-help
  Show this screen.

/hiring-prepare    — Pre-interview briefing for any candidate
/hiring-debrief    — Debrief synthesis, prep, and decision support
/hiring-admin      — Add functions, run backtests, validate plugin
```

---

## Step 6 — Quick Tips

Show context-sensitive tips based on config state:

- If no assessments in cohort yet:
  ```
  Tip: Run /hiring-evaluate --confirm on your first candidate to walk through each gate.
  ```

- If ATS not configured:
  ```
  Tip: To enable Greenhouse integration, add GREENHOUSE_API_KEY to your shell profile and re-run /hiring-setup.
  ```

- If calibration examples directory is empty or missing:
  ```
  Tip: Add past assessments to ~/.hiring-assistant/functions/{function}/examples/{level}/ to improve calibration accuracy.
  ```

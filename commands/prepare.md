# /hiring-prepare

Generates a pre-interview briefing for a candidate — candidate background, suggested probing questions per competency, and level bar reminder.

**Usage:** `/hiring-prepare [--auto]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first
2. **At least one function configured:** `config.functions` is non-empty

---

## Step 1 — Candidate Discovery

Dispatch: `agents/interview-router.md`

```xml
<config>{~/.hiring-assistant/config.yaml}</config>
<search_window>30</search_window>
```

Receive: `<routing_result>` with candidate, level, function, type.

---

## Step 2 — Guidelines

Dispatch: `agents/guideline-loader.md`

```xml
<routing_result>{from Step 1}</routing_result>
<config>{~/.hiring-assistant/config.yaml}</config>
<org_config>{~/.hiring-assistant/org-config.yaml if present}</org_config>
```

Receive: `<guideline_load_result>` with guidelines and function config.

---

## Step 3 — Briefing

Dispatch: `agents/prepare-briefing.md`

```xml
<routing_result>{from Step 1}</routing_result>
<guideline_load_result>{from Step 2}</guideline_load_result>
```

Receive: completed briefing markdown.

---

## Step 4 — Output

Print the full briefing to screen.

Offer to save:
```
Briefing ready for {Candidate Name} ({Level} {Interview Type}).
Save to ~/.hiring-assistant/candidates/{name}/briefing-{type}.md? [Yes / No]
```

---

## Flags

| Flag | Effect |
|------|--------|
| `--auto` | Skip confirmations; auto-pick most recent candidate; save briefing without asking |

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| Config not found | "Run /hiring-setup first." |
| No calendar match | Falls back to manual entry in interview-router |
| Guidelines missing | Generates briefing with competency structure only (no level bar or must-identify insights) |

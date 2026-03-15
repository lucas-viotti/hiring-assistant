# /hiring-admin

Admin command for function owners and power users. Provides four subcommands: validate, backtest, clean, and sync.

**Usage:** `/hiring-admin [validate|backtest|clean|sync]`

---

## Pre-flight

Before running, check:

1. **Config exists:** `~/.hiring-assistant/config.yaml` — if missing, trigger `/hiring-setup` first

---

## Subcommand Selection

If no subcommand is given, show the menu:

```
/hiring-admin — Admin Tools

[1] validate   — Check plugin structure and config files
[2] backtest   — Run golden-set backtests against saved assessments
[3] clean      — Scan candidate files for retention window expiry
[4] sync       — Pull latest guidelines from configured repos

Which would you like to run?
```

---

## Subcommand: validate

Run: `tests/validate-plugin.sh` (structure check only, no golden-set comparison)

Show full output. On success:
```
✅ Plugin structure valid.
```

On failure:
```
❌ Validation failed. See output above.
```

---

## Subcommand: backtest

Run: `tests/validate-plugin.sh --golden-set`

This runs the backtest suite against the saved golden-set files in `tests/backtests/golden-set/`.

Show pass/fail per test case. On completion:
```
Backtest complete: {N} passed, {M} failed.
{List failed cases with path if any}
```

If no golden-set files exist:
```
No golden-set files found at tests/backtests/golden-set/.
Add a golden-set assessment to enable backtests.
See tests/backtests/README.md for instructions.
```

---

## Subcommand: clean

Scan `~/.hiring-assistant/candidates/` for files older than the configured `retention_days`.

Read `retention_days` from `~/.hiring-assistant/config.yaml` (or `org-config.yaml`).

If `retention_days == 0`:
```
Retention cleanup is disabled (retention_days: 0). No action taken.
```

Otherwise, list all candidate directories and their most recent file modification dates.

Flag directories where all files are older than `retention_days` days:

```
The following candidate directories are past the retention window ({N} days):

  ~/.hiring-assistant/candidates/jane-doe/
    Last modified: {date} ({X} days ago)
    Files: assessment-behavioral.md, transcript.md

  ~/.hiring-assistant/candidates/john-smith/
    Last modified: {date} ({X} days ago)
    Files: assessment-business-case.md

Delete these directories? [Yes / No / Review each]
```

If user confirms → delete flagged directories and confirm.

---

## Subcommand: sync

Read `guidelines_repo` from `~/.hiring-assistant/functions/{function}/config.yaml` for each configured function.

For each function with a guidelines repo configured:
- Check if `~/.hiring-assistant/guidelines/{function}/` exists and is a git repo
- If yes → run `git -C ~/.hiring-assistant/guidelines/{function}/ pull`
- Show pull output

```
Syncing guidelines for product-ops...
  Already up to date.  (or: N commits pulled)

Sync complete.
```

If a guidelines directory is not a git repo:
```
⚠️ ~/.hiring-assistant/guidelines/product-ops/ is not a git repository.
Re-run /hiring-setup to reconfigure guidelines access.
```

---

## Error Recovery

| Error | Recovery |
|-------|---------|
| validate-plugin.sh not found | "Run from the hiring-assistant plugin directory." |
| No candidate directories | "No candidate data found at ~/.hiring-assistant/candidates/" |
| Git pull fails (auth) | Show git error; suggest `gh auth status` to check credentials |
| Config not found | "Run /hiring-setup first." |

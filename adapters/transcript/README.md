# Transcript Adapters

Extensible registry for acquiring interview transcripts. Each transcript source = one YAML adapter. Drop a new `.yaml` (and optional paired `.md`) to support a new recording platform — no core plugin changes required.

---

## How It Works

When `/hiring-evaluate` reaches Gate 5 (transcript acquisition), the `transcript-fetcher` agent:

1. Reads the `default_adapter` from `~/.hiring-assistant/config.yaml`
2. Loads the corresponding `.yaml` from this directory
3. Executes the acquisition method (`clipboard`, `file`, or `mcp`)
4. Runs the raw transcript through the enrichment pipeline (`transcript-enrichment/`)
5. Saves the enriched result to `~/.hiring-assistant/candidates/{candidate-name}/transcript.md`

If no transcript is available (async interview type, or user skips), the evaluation proceeds with notes only and confidence is flagged automatically.

---

## YAML Schema

```yaml
name: ""                    # Unique identifier (e.g., "brighthire-clipboard")
description: ""             # Short description for /hiring-setup display
platform: ""                # Display name of the platform (e.g., "BrightHire")
method: ""                  # Acquisition method — one of:
                            #   "clipboard"   → Prompt user to paste transcript text
                            #   "file"        → Prompt user to provide a file path
                            #   "mcp"         → Fetch automatically via MCP tool
mcp: ""                     # Required MCP server name (if method: "mcp")
tools: []                   # MCP tools used (for documentation; helps /hiring-setup verify availability)
instructions: ""            # Path to paired .md with fetch/parse logic
file_extensions: []         # Expected file extensions (if method: "file")
file_prompt: ""             # Instructions shown to user (if method: "file")
clipboard_prompt: ""        # Instructions shown to user (if method: "clipboard")
output: "markdown"          # Output format after parsing (always "markdown")
```

---

## Adding a New Adapter

1. Create `{platform-name}.yaml` in this directory following the schema above
2. *(Optional)* Create `{platform-name}.md` with parse/extraction logic
3. If `method: "mcp"`, document the MCP server in `tools` so `/hiring-setup` can check availability and suggest installation

**Example:** To add Otter.ai support:
```yaml
name: "otter-ai"
description: "Fetch transcript from Otter.ai via file export"
platform: "Otter.ai"
method: "file"
file_extensions: [".txt", ".docx"]
file_prompt: "Export your transcript from Otter.ai (File → Export → Text) and provide the path."
instructions: "otter-ai.md"
output: "markdown"
```

---

## Included Adapters

| Adapter | Method | Status |
|---------|--------|--------|
| BrightHire (clipboard) | `clipboard` | ✅ Active |
| Manual file | `file` | ✅ Active |

The **manual-file** adapter accepts `.md`, `.txt`, `.docx`, and `.vtt` files — covering exports from Zoom, Microsoft Teams, Granola, Otter.ai, and other recording platforms that export transcripts as files.

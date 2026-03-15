# Material Readers

Extensible registry for extracting content from candidate materials. Each file type = one YAML adapter. Drop a new `.yaml` (and optional paired `.md`) to add support for a new format — no core plugin changes required.

---

## How It Works

When `/hiring-evaluate` or `/hiring-prepare` encounters candidate materials, the `materials-orchestrator` agent:

1. Reads all `.yaml` files in this directory
2. Matches each material file/URL against `match.url_patterns` and `match.file_extensions`
3. Dispatches to the matched reader using the `mcp` field
4. Passes extraction instructions from the paired `.md` file (if present)

---

## YAML Schema

```yaml
name: ""                    # Unique identifier (e.g., "google-slides")
match:
  url_patterns:             # URL substrings that trigger this reader (OR logic)
    - ""                    # e.g., "docs.google.com/presentation"
  file_extensions:          # File extensions that trigger this reader (OR logic)
    - ""                    # e.g., ".pptx"
mcp: ""                     # Dispatch method — one of:
                            #   "native"      → Read tool (plain files)
                            #   MCP server    → Name of MCP server (e.g., "google-workspace")
                            #   "agent"       → Spawn a subagent (requires `agent` field)
                            #   "cli"         → Run a shell command (requires `cli` field)
agent: ""                   # Subagent name, if mcp: "agent" (e.g., "pdf-extractor")
cli: ""                     # Command template, if mcp: "cli" (e.g., "python3 {plugin-dir}/scripts/extract-pdf.py {file}")
tools: []                   # MCP tools used (for documentation; helps orchestrator verify availability)
output: ""                  # Output format: "markdown" | "markdown + images" | "text"
instructions: ""            # Path to paired .md file with extraction prompt/logic
```

---

## Adding a New Reader

1. Create `{format-name}.yaml` in this directory following the schema above
2. *(Optional)* Create `{format-name}.md` with extraction instructions — the orchestrator will pass this as the prompt when dispatching
3. If `mcp` requires an MCP server, document it in the `tools` field so `/hiring-setup` can check availability

**Example:** To add Notion support:
```yaml
name: "notion"
match:
  url_patterns:
    - "notion.so"
    - "notion.site"
mcp: "notion"
tools: ["notion_get_page", "notion_get_blocks"]
output: "markdown"
instructions: "notion.md"
```

---

## Phase 1 Readers

| Reader | File | Status |
|--------|------|--------|
| PDF | `pdf.yaml` | ✅ Active |

## Planned Readers (Phase 2)

| Reader | Trigger | MCP / Method |
|--------|---------|-------------|
| Google Slides | `docs.google.com/presentation` | `google-workspace` |
| Google Docs | `docs.google.com/document` | `google-workspace` |
| PowerPoint | `.pptx` | `native` |
| Word | `.docx` | `native` |
| Google Sheets | `docs.google.com/spreadsheets` | `google-workspace` |
| Excel | `.xlsx` | `native` |
| Miro board | `miro.com/app/board` | `miro-mcp` |
| Figma | `figma.com/file` | `figma` |
| GitHub repo | `github.com/{org}/{repo}` | `agent` → `code-explorer` |
| Jupyter Notebook | `.ipynb` | `native` |

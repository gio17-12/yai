## Description
Performs a web search using the Brave Search API and returns formatted markdown results (titles, URLs, snippets) suitable for LLM comprehension.

## How to use it
<!-- SYNTAX:START -->
`bot/tools/web-search/run "<query>" [--count N] [--dry-run]`
<!-- SYNTAX:END -->

### Arguments & Flags
- `query` *(string, required)*: The search query string (enclosed in quotes if it contains spaces).
- `--count`, `-n` *(integer, optional)*: Number of results to return (default: `5`, range: `1..20`).
- `--dry-run`, `-d` *(flag, optional)*: Simulates execution without making external network calls, inspecting parameters, effects, and output preview while recording telemetry.

### Configuration & Secrets
Requires a Brave Search API key. You can provide it in two ways:
1. Export in shell: `export BRAVE_API_KEY="BSA..."`
2. Define in `main/.env` (or root `.env`):
   ```bash
   BRAVE_API_KEY="BSA..."
   ```

## Examples
```bash
# Real execution
bot/tools/web-search/run "latest quantum computing developments"
bot/tools/web-search/run "python asyncio best practices" --count 3

# Dry-run simulation
bot/tools/web-search/run "test query" --dry-run
```

## Details

### 1. Inputs
- `query` string and optional `count` integer.
- `BRAVE_API_KEY` from environment or `.env`.

### 2. Outputs
- **STDOUT**: Clean Markdown results with headers `### <index>. <title>`, `**URL:** <url>`, and descriptive snippets.
- **STDERR**: Status messages and error alerts.
- **Exit Code**: `0` on success, `1` on missing key, invalid parameters, or API failure.

### 3. Side Effects
- **Network**: Exactly 1 outbound HTTPS GET request to `https://api.search.brave.com/res/v1/web/search`. Blocked when running with `--dry-run`.
- **Filesystem**: 0 mutations (no files created or deleted). Read-only access to `.env` for credential loading. When `--dry-run` is invoked, appends execution telemetry to `history.jsonl`.
- **Subprocesses**: None.

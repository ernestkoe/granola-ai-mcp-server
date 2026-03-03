# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP server that exposes Granola.ai meeting data (notes, transcripts, participants) to AI assistants. It reads directly from Granola's local cache file (`~/Library/Application Support/Granola/cache-v4.json`, falling back to `cache-v3.json`) — no API keys needed, macOS only.

## Commands

```bash
# Run the server
uv run granola-mcp-server

# Run unit tests (synthetic cache data)
uv run python test_server.py

# Run integration tests (requires Granola desktop app with real cache)
uv run python test_real_cache.py

# Install dependencies
uv sync
```

## Architecture

**Single-server design** — almost all logic lives in `granola_mcp_server/server.py` (~760 lines):

- **`GranolaMCPServer`** class: Handles cache loading, data parsing, tool dispatch, and response formatting
- **`models.py`**: Pydantic models (`CacheData`, `MeetingMetadata`, `MeetingDocument`, `MeetingTranscript`) — thin data containers only
- **Entry point**: `main()` at bottom of `server.py`, registered as `granola-mcp-server` CLI via pyproject.toml

### MCP Tools (5 total, defined in `_setup_handlers`)

| Tool | Purpose |
|------|---------|
| `search_meetings` | Keyword search across titles, participants, transcripts with relevance scoring |
| `get_meeting_details` | Full meeting info by ID (notes, participants, time) |
| `get_meeting_transcript` | Raw transcript with speaker attribution |
| `get_meeting_documents` | Document/notes content for a meeting |
| `analyze_meeting_patterns` | Aggregate analysis: `topics`, `participants`, or `frequency` |

### Cache Parsing Pipeline

Granola's cache has two format versions. In v3 (`cache-v3.json`), the `cache` key contains a JSON **string** (double-encoded). In v4 (`cache-v4.json`), the `cache` key is a **dict** directly. The server auto-detects which file exists (preferring v4) and handles both formats. The parsing flow:

1. `_load_cache()` → reads file, double-parses JSON
2. `_parse_cache_data()` → transforms raw Granola format into Pydantic models
3. `_ensure_cache_loaded()` → lazy-loads before each tool call

**Document content extraction** uses a 4-tier fallback: `notes_plain` → `notes_markdown` → structured notes → `documentPanels` (controlled by `GRANOLA_PARSE_PANELS` env var, default on).

### Timezone Handling

Auto-detects system timezone via `time.tzname` with US timezone mappings. Falls back to `America/New_York`. All stored times are UTC; converted to local for display via `_format_local_time()`.

## Environment Variables

- `GRANOLA_PARSE_PANELS` — set to `"0"` to disable panel parsing fallback (default: `"1"`)
- `TZ` — override auto-detected timezone

## Release Process

Uses **python-semantic-release** via GitHub Actions on push to `main`. Conventional commit messages drive version bumps. Releases are published to [PyPI](https://pypi.org/project/granola-mcp-server/) via trusted publishing. Config lives in `pyproject.toml` under `[tool.semantic_release]`.

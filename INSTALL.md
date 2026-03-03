# Installation Guide

## Quick Setup

1. **Clone/Download the project**
   ```bash
   cd /Users/ernestkoe/Projects/granola-ai-mcp
   ```

2. **Create and activate virtual environment**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install mcp pydantic
   ```

4. **Test the server**
   ```bash
   python test_server.py
   ```

## Claude Desktop Integration

1. **Find your Claude Desktop config file**
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
   - Linux: `~/.config/Claude/claude_desktop_config.json`

2. **Add the server configuration**
   ```json
   {
     "mcpServers": {
       "granola": {
         "command": "python",
         "args": ["/Users/ernestkoe/Projects/granola-ai-mcp/run_server.py"],
         "env": {
           "PYTHONPATH": "/Users/ernestkoe/Projects/granola-ai-mcp/.venv/lib/python3.13/site-packages"
         }
       }
     }
   }
   ```

   Or if you have `uv` installed:
   ```json
   {
     "mcpServers": {
       "granola": {
         "command": "uv",
         "args": ["--directory", "/Users/ernestkoe/Projects/granola-ai-mcp", "run", "python", "run_server.py"],
         "env": {}
       }
     }
   }
   ```

3. **Restart Claude Desktop**

## Verify Installation

In Claude Desktop, try asking:
- "Search for meetings about standup"
- "List available meeting tools"
- "Show me my recent meetings"

## Troubleshooting

### Cache File Not Found
If you get errors about the cache file, ensure Granola.ai is installed and has created meetings. The server auto-detects the cache at:
```
~/Library/Application Support/Granola/cache-v4.json  (current)
~/Library/Application Support/Granola/cache-v3.json  (legacy)
```

### Python Path Issues  
Make sure the virtual environment is properly referenced in your Claude config. You can also install system-wide with:
```bash
pip3 install --user mcp pydantic
```

### Permission Errors
Ensure the cache file is readable:
```bash
ls -la ~/Library/Application\ Support/Granola/cache-v*.json
```
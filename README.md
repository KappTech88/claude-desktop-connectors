# 🔌 Claude Desktop Local MCP Connectors

Custom-built local MCP (Model Context Protocol) servers for Claude Desktop on Windows. Each project is a standalone, self-contained connector you can clone and run independently — no bloated monorepo installs required.

Built by [Bryant Quiovers](https://github.com/YOUR_USERNAME) · Kapp Labs

---

## What Is This?

This repository is a collection of **custom MCP servers** designed to run locally alongside [Claude Desktop](https://claude.ai/download) on Windows. Each folder is a self-contained project with its own Python virtual environment, OAuth credentials flow, and `server.py` entrypoint. They extend Claude Desktop's capabilities beyond the built-in connectors — giving it direct access to your Gmail accounts, Google Drive files, and more.

Most were built from scratch to solve real workflow problems. Some are forks of existing tools adapted to run as local MCP servers for Claude Desktop (noted in the table below).

---

## Available Connectors

| Connector | Description | Auth | Status |
|---|---|---|---|
| [`gmail-multi`](./gmail-multi) | Multi-account Gmail server — manage up to 10+ Gmail accounts from a single MCP server with fuzzy account matching, full CRUD (search, read, send, draft, label, trash), and per-account OAuth token caching. | OAuth 2.0 (per account) | ✅ Stable |
| [`gdrive-mcp`](./gdrive-mcp) | Google Drive server — search, read, upload, download, move, trash, delete, and create folders across your Drive. Full `drive` scope for complete file management. | OAuth 2.0 | ✅ Stable |
| [`ai-research-skills`](./ai-research-skills) | Forked from [Orchestra Research's AI-Research-SKILLs](https://github.com/Orchestra-Research/AI-Research-SKILLs) and adapted to run as a local MCP server for Claude Desktop. 91 research skills across 22 categories — literature review, data analysis, prompt engineering, fine-tuning, RAG, agents, and more. Originally built as a Claude Code plugin. | None | ✅ Stable |

> More connectors will be added as they're built. Contributions welcome.

---

## Requirements

- **Windows 10/11** (tested on Windows 11)
- **Python 3.10+** (each project creates its own venv — no system-wide installs needed)
- **Claude Desktop** installed and running
- **Google Cloud Console** project with OAuth 2.0 credentials (only for Google-based connectors like `gmail-multi` and `gdrive-mcp`)

> **Note:** Not all connectors require OAuth or Google credentials. Check each connector's README for its specific requirements.

---

## Quick Start (Any Connector)

### 1. Clone Only the Connector You Need

You don't need to download the entire repository. Use Git's **sparse checkout** to grab just the project folder you want:

```bash
# Create and enter a new directory
mkdir claude-connectors && cd claude-connectors

# Initialize a git repo and add this repository as a remote
git init
git remote add origin https://github.com/YOUR_USERNAME/claude-desktop-mcp-connectors.git

# Enable sparse checkout
git sparse-checkout init --cone

# Pull ONLY the connector you want (e.g., gmail-multi)
git sparse-checkout set gmail-multi
git pull origin main
```

This downloads **only** the `gmail-multi` folder and nothing else.

#### Want multiple connectors?

```bash
git sparse-checkout set gmail-multi gdrive-mcp
git pull origin main
```

### 2. Alternative: Download a Single Folder as ZIP

If you don't want to use Git at all, you can download a specific folder directly from GitHub:

**Option A — Use the `download-directory` tool:**

Go to [download-directory.github.io](https://download-directory.github.io/), paste the URL of the specific folder from this repo, and it will generate a ZIP of just that folder.

Example URL to paste:
```
https://github.com/YOUR_USERNAME/claude-desktop-mcp-connectors/tree/main/gmail-multi
```

**Option B — Use `svn export` (if you have Subversion installed):**

```bash
svn export https://github.com/YOUR_USERNAME/claude-desktop-mcp-connectors/trunk/gmail-multi
```

**Option C — Use the GitHub CLI:**

```bash
# Clone the repo shallowly and filter to just the folder you need
gh repo clone YOUR_USERNAME/claude-desktop-mcp-connectors -- --depth 1 --filter=blob:none --sparse
cd claude-desktop-mcp-connectors
git sparse-checkout set gmail-multi
```

### 3. Set Up the Connector

Each connector folder has its own `README.md` with specific setup instructions, but the general pattern is:

```bash
cd gmail-multi  # or gdrive-mcp, etc.

# Create a Python virtual environment
python -m venv venv

# Install dependencies
venv\Scripts\pip.exe install -r requirements.txt

# Add your OAuth credentials
# (copy your credentials.json from Google Cloud Console into the project folder)

# Run the auth script to generate tokens
venv\Scripts\python.exe auth_accounts.py

# Add the server to your Claude Desktop config
# (see "Connecting to Claude Desktop" below)
```

### 4. Connect to Claude Desktop

Add the server entry to your `claude_desktop_config.json`:

**Config location:**
```
C:\Users\<YOUR_USERNAME>\AppData\Roaming\Claude\claude_desktop_config.json
```

**Example entry (gmail-multi):**
```json
{
  "mcpServers": {
    "gmail-multi": {
      "command": "C:\\path\\to\\gmail-multi\\venv\\Scripts\\python.exe",
      "args": ["C:\\path\\to\\gmail-multi\\server.py"]
    }
  }
}
```

After saving the config, **fully restart Claude Desktop** (quit from the system tray, not just close the window).

---

## Project Structure

**Google OAuth connectors** (`gmail-multi`, `gdrive-mcp`) follow this layout:

```
connector-name/
├── server.py              # MCP server entrypoint
├── auth_accounts.py       # OAuth authentication script
├── config.json            # Connector-specific configuration
├── credentials.json       # OAuth client credentials (NOT committed — see .gitignore)
├── requirements.txt       # Python dependencies
├── tokens/                # OAuth tokens (NOT committed — generated locally)
│   └── .gitkeep
└── README.md              # Connector-specific setup guide
```

**Forked / non-OAuth connectors** (`ai-research-skills`) have their own structure — check each project's README for details.

---

## Important Notes

- **Never commit `credentials.json` or `tokens/*.pickle`** — these contain your OAuth secrets and session tokens. The `.gitignore` in this repo excludes them, but double-check before pushing.
- **Always use the venv Python path** in your Claude Desktop config — never rely on system PATH.
- **Restart Claude Desktop fully** after any config change (quit from system tray).
- **Windows-specific:** Do NOT use PowerShell's `Set-Content` to write JSON or Python files — it injects a UTF-8 BOM that breaks `json.load()`. Use a proper text editor or write files programmatically.
- **Each connector is independent** — they don't share venvs, tokens, or configs. You can run any combination simultaneously.

---

## Troubleshooting

**Server not showing in Claude Desktop?**
Check that `claude_desktop_config.json` has valid JSON (no trailing commas), verify the Python path exists, and restart Claude Desktop from the system tray.

**Token expired / auth error?**
Delete the relevant `.pickle` file from `tokens/`, run `auth_accounts.py` again, and restart Claude Desktop.

**"Access denied" or scope errors?**
Your OAuth token may have been created with limited scopes. Delete the token, update scopes in `server.py` and `auth_accounts.py` if needed, re-authenticate, and restart.

**PowerShell wrote broken files?**
If you used PowerShell `Set-Content` and things broke, the file likely has a BOM. Re-save it with a normal editor (VS Code, Notepad++) in UTF-8 without BOM.

---

## Contributing

Found a bug? Want to add a connector? PRs are welcome. Each new connector should follow the project structure above and include its own `README.md` with setup instructions.

---

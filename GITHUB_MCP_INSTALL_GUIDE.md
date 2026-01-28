# GitHub MCP Server Installation Guide

This document provides comprehensive instructions for installing and configuring the GitHub Model Context Protocol (MCP) Server in Claude applications.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Claude Code CLI Setup](#claude-code-cli-setup)
3. [Claude Desktop Setup](#claude-desktop-setup)
4. [Verification](#verification)
5. [Troubleshooting](#troubleshooting)
6. [Important Notes](#important-notes)

## Prerequisites

Before starting, ensure you have:

- Claude Code CLI or Claude Desktop (latest version)
- GitHub Personal Access Token (PAT)
- For local setup: Docker installed and running
- (Recommended) Open Claude in your project directory for best experience and clear configuration scope

## Claude Code CLI Setup

### Storing Your PAT Securely

We strongly recommend storing your PAT in an environment variable or `.env` file instead of hardcoding it in commands:

```bash
echo "GITHUB_PAT=your_actual_token_here" > .env
```

### Remote Server Setup (Streamable HTTP)

> **Note**: For Claude Code versions 2.1.1 and newer, use the `add-json` command format below. For older versions, see the legacy command format.

Run the following command in the terminal (not in Claude Code CLI):

```bash
claude mcp add-json github '{"type":"http","url":"https://api.githubcopilot.com/mcp","headers":{"Authorization":"Bearer YOUR_GITHUB_PAT"}}'
```

With an environment variable:

```bash
claude mcp add-json github '{"type":"http","url":"https://api.githubcopilot.com/mcp","headers":{"Authorization":"Bearer '"$(grep GITHUB_PAT .env | cut -d '=' -f2)"'}}'
```

#### About the --scope flag (optional):
Use this to specify where the configuration is stored:

- `local` (default): Available only to you in the current project (was called project in older versions)
- `project`: Shared with everyone in the project via .mcp.json file
- `user`: Available to you across all projects (was called global in older versions)

Example: Add `--scope user` to the end of the command to make it available across all projects:

```bash
claude mcp add-json github '{"type":"http","url":"https://api.githubcopilot.com/mcp","headers":{"Authorization":"Bearer YOUR_GITHUB_PAT"}}' --scope user
```

After configuration, restart Claude Code and run:

```bash
claude mcp list
```

### Local Server Setup (Docker required)

#### With Docker
Run the following command in the terminal (not in Claude Code CLI):

```bash
claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_GITHUB_PAT -- docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server
```

With an environment variable:

```bash
claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=$(grep GITHUB_PAT .env | cut -d '=' -f2) -- docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server
```

#### With a Binary (no Docker)
1. Download release binary
2. Add to your PATH
3. Run:
```bash
claude mcp add-json github '{"command": "github-mcp-server", "args": ["stdio"], "env": {"GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_GITHUB_PAT"}}'
```

After configuration, restart Claude Code and run:

```bash
claude mcp list
```

## Claude Desktop Setup

> **⚠️ Note**: Some users have reported compatibility issues with Claude Desktop and Docker-based MCP servers. We're investigating. If you experience issues, try using another MCP host, while we look into it!

### Prerequisites
- Claude Desktop installed (latest version)
- GitHub Personal Access Token
- Docker installed and running

> **Note**: Claude Desktop supports MCP servers that are both local (stdio) and remote ("connectors"). Remote servers can generally be added via Settings → Connectors → "Add custom connector". However, the GitHub remote MCP server requires OAuth authentication through a registered GitHub App (or OAuth App), which is not currently supported. Use the local Docker setup instead.

### Configuration File Location
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

### Local Server Setup (Docker)

Add this code block to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_GITHUB_PAT"
      }
    }
  }
}
```

### Manual Setup Steps

1. Open Claude Desktop
2. Go to Settings → Developer → Edit Config
3. Paste the code block above in your configuration file
4. If you're navigating to the configuration file outside of the app:
   - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
   - Windows: `%APPDATA%\Claude\claude_desktop_config.json`
5. Open the file in a text editor
6. Paste one of the code blocks above, based on your chosen configuration (remote or local)
7. Replace `YOUR_GITHUB_PAT` with your actual token or `$GITHUB_PAT` environment variable
8. Save the file
9. Restart Claude Desktop

## Verification

After installation, verify the setup by running:

```bash
claude mcp list
claude mcp get github
```

## Legacy Versions Support

If you're using Claude Code version 2.1.0 or earlier, use this legacy command format:

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp -H "Authorization: Bearer YOUR_GITHUB_PAT"
```

With an environment variable:

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp -H "Authorization: Bearer $(grep GITHUB_PAT .env | cut -d '=' -f2)"
```

### Windows (PowerShell)
If you see missing required argument 'name', put the server name immediately after claude mcp add:

```powershell
$pat = "YOUR_GITHUB_PAT"
claude mcp add github --transport http https://api.githubcopilot.com/mcp/ -H "Authorization: Bearer $pat"
```

## Troubleshooting

### Authentication Failed:
- Verify PAT has repo scope
- Check token hasn't expired

### Remote Server:
- Verify URL: https://api.githubcopilot.com/mcp

### Docker Issues (Local Only):
- Ensure Docker Desktop is running
- Try: `docker pull ghcr.io/github/github-mcp-server`
- If pull fails: `docker logout ghcr.io` then retry

### Server Not Starting / Tools Not Showing:
- Run `claude mcp list` to view currently configured MCP servers
- Validate JSON syntax
- If using an environment variable to store your PAT, make sure you're properly sourcing your PAT using the environment variable
- Restart Claude Code and check /mcp command
- Delete the GitHub server by running `claude mcp remove github` and repeating the setup process with a different method
- Make sure you're running Claude Code within the project you're currently working on to ensure the MCP configuration is properly scoped to your project
- Check logs:
  - Claude Code: Use /mcp command
  - Claude Desktop: `ls ~/Library/Logs/Claude/` and `cat ~/Library/Logs/Claude/mcp-server-*.log` (macOS) or `%APPDATA%\Claude\logs\` (Windows)

## Important Notes

- The npm package `@modelcontextprotocol/server-github` is deprecated as of April 2025
- Remote server requires Streamable HTTP support (check your Claude version)
- For Claude Code configuration scopes, see the --scope flag documentation in the Remote Server Setup section

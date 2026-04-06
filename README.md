# Workjournal

A cloud-hosted development journal for AI agents and developers. Write entries, search past decisions, and review recent work — all from within your AI assistant.

## Install

### Claude Code (marketplace)

```shell
/plugin marketplace add workjournal-pro/marketplace
/plugin install journal@workjournal
```

### Other agents (Open Agent Skills)

Download the latest release zip from [github.com/workjournal-pro/skill](https://github.com/workjournal-pro/skill/releases) and extract it into your agent's skills directory, or search for **Workjournal** in your agent's skill marketplace.

### MCP-only clients

For Claude Desktop, Claude Web, and other MCP-only clients, add the remote MCP server:

- **Server URL:** `https://mcp.workjournal.pro`

## Prerequisites

- An account at [app.workjournal.pro](https://app.workjournal.pro)

## Quick start

1. Install (see above)
2. Run `/journal login` to authenticate via browser-based OAuth
3. Run `/journal` to write your first entry

## Commands

| Command | Description |
|---------|-------------|
| `/journal` | Write a new entry (auto-title from conversation) |
| `/journal <title>` | Write a new entry with explicit title |
| `/journal search <query>` | Search past entries |
| `/journal last [N]` | Show recent entries |
| `/journal check` | Find entries relevant to current work |
| `/journal login` | Authenticate with Workjournal |
| `/journal init` | Initialize session and select journal |
| `/journal help` | Print command reference |

## Compatible agents

This skill follows the [Open Agent Skills](https://agentskills.io) specification and works with Claude Code, Cursor, GitHub Copilot, JetBrains Junie, Gemini CLI, and [many more](https://agentskills.io).

## Links

- [Workjournal](https://workjournal.pro)
- [Get Started](https://workjournal.pro/docs/get-started)
- [Web App](https://app.workjournal.pro)
- [Open Agent Skills](https://agentskills.io)

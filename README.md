# Workjournal

A cloud-hosted development journal for AI agents and developers. Write entries, search past decisions, and review recent work — all from within your AI assistant.

## Install

### Claude Code (marketplace)

```shell
/plugin marketplace add workjournal-pro/marketplace
/plugin install workjournal@workjournal
```

> Renamed from `journal@workjournal` in v0.10.0. If you previously installed under the old name, run `/plugin uninstall journal@workjournal` first.

### Codex (marketplace)

```shell
codex plugin marketplace add workjournal-pro/marketplace
codex plugin add workjournal@workjournal
```

Start a new Codex thread after installation so the Workjournal skill is loaded. Ask Codex to log completed work, search past decisions, or review recent entries. On first use, ask Codex to log in to Workjournal; it drives the same browser-based OAuth flow as the CLI.

### Other agents (Open Agent Skills)

Download the latest release zip from [github.com/workjournal-pro/skill](https://github.com/workjournal-pro/skill/releases) and extract it into your agent's skills directory, or search for **Workjournal** in your agent's skill marketplace.

### MCP-only clients

For Claude Desktop, Claude Web, and other MCP-only clients, add the remote MCP server:

- **Server URL:** `https://mcp.workjournal.pro`

## More plugins in this marketplace

Alongside the Workjournal journal plugin, this marketplace hosts two standalone workflow skills. They install independently — you don't need the Workjournal account or login to use them.

### `wj-gh` — GitHub change workflow

Drives a change from branch to merged PR: scope → optional issue → branch → commit (with tests) → open PR → watch CI → respond to a code-review bot → fix mechanical failures → reply inline → merge. Provider-agnostic; treats CodeRabbit as an optional integration.

```shell
# Claude Code
/plugin install wj-gh@workjournal

# Codex
codex plugin add wj-gh@workjournal
```

Open Agent Skills: [github.com/workjournal-pro/wj-gh](https://github.com/workjournal-pro/wj-gh/releases).

### `prime` — session primer

Lists tracked and changed files, surfaces the last few journal entries (Workjournal or devjournal), and reports current branch status (open PR, merged, or in-progress) at the start of a session.

```shell
# Claude Code
/plugin install prime@workjournal

# Codex
codex plugin add prime@workjournal
```

Open Agent Skills: [github.com/workjournal-pro/prime](https://github.com/workjournal-pro/prime/releases).

## Prerequisites

- **Workjournal plugin:** an account at [app.workjournal.pro](https://app.workjournal.pro). The `/workjournal login` flow authenticates against it.
- **`wj-gh` and `prime`:** no Workjournal account or login — they only need the `git` and `gh` CLIs available in your environment.

## Quick start (Workjournal plugin)

1. Install (see above)
2. Run `/workjournal login` to authenticate via browser-based OAuth
3. Run `/workjournal` to write your first entry

`wj-gh` and `prime` work as soon as they're installed — invoke `/wj-gh help` or `/prime` directly.

## Commands

| Command | Description |
|---------|-------------|
| `/workjournal` | Write a new entry (auto-title from conversation) |
| `/workjournal <title>` | Write a new entry with explicit title |
| `/workjournal search <query>` | Search past entries |
| `/workjournal last [N]` | Show recent entries |
| `/workjournal check` | Find entries relevant to current work |
| `/workjournal login` | Authenticate with Workjournal |
| `/workjournal help` | Print command reference |

## Compatible agents

This skill follows the [Open Agent Skills](https://agentskills.io) specification and works with Claude Code, Cursor, GitHub Copilot, JetBrains Junie, Gemini CLI, and [many more](https://agentskills.io).

## Links

- [Workjournal](https://workjournal.pro)
- [Get Started](https://workjournal.pro/docs/get-started)
- [Web App](https://app.workjournal.pro)
- [Open Agent Skills](https://agentskills.io)

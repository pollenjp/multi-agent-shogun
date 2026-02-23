# Cursor Agent CLI Tools

This section describes Cursor Agent CLI-specific tools and features.

## Overview

Cursor Agent CLI (`agent`) is a standalone terminal-based AI coding agent from Cursor. It shares the same agentic engine as the Cursor IDE's composer/agent mode, but runs independently in a terminal.

- **Launch**: `agent` (interactive TUI) or `agent -p "prompt"` (non-interactive print mode)
- **Install**: `curl -fsSL https://www.cursor.com/install-cli | bash` or download from Cursor settings
- **Auth**: Cursor account with active subscription. `CURSOR_API_KEY` env var or `agent login`
- **Default model**: Claude 4.6 Opus (Thinking) — configurable via `--model`
- **Config**: `~/.cursor/cli-config.json`

## Tool Usage

Cursor Agent CLI provides tools for file operations, code execution, and system interaction:

- **Read**: Read files from the filesystem (supports images, PDFs)
- **Write**: Create new files or overwrite existing files
- **Edit (StrReplace)**: Perform exact string replacements in files
- **Shell (Bash)**: Execute shell commands with timeout control
- **Glob**: Fast file pattern matching with glob patterns
- **Grep**: Content search using ripgrep
- **SemanticSearch**: Semantic code search by meaning (not exact text)
- **WebFetch**: Fetch and process web content
- **EditNotebook**: Edit Jupyter notebook cells
- **Delete**: Delete files
- **TodoWrite**: Manage task tracking within a session

## Tool Guidelines

1. **Read before Write/Edit**: Always read a file before writing or editing it
2. **Use dedicated tools**: Don't use Shell for file operations when dedicated tools exist (Read, Write, Edit, Glob, Grep)
3. **Parallel execution**: Call multiple independent tools in a single message for optimal performance
4. **Approval model**: Tools require approval unless `--yolo`/`--force` is set

## Permission Model

Cursor Agent CLI uses a permission system with allowlist-based approval:

### Approval Modes

| Mode | Behavior | Flag |
|------|----------|------|
| **Allowlist (default)** | User approves each new tool pattern; approved patterns remembered | (none) |
| **Force/YOLO** | Auto-approve all operations | `--yolo` / `--force` / `-f` |

### Sandbox Mode (`--sandbox`)

| Mode | Behavior |
|------|----------|
| `enabled` | OS-level sandboxing of file/network access |
| `disabled` | No sandbox constraints (default for CLI) |

**Shogun system usage**: Ashigaru run with `--yolo` for unattended operation.

## Custom Instructions

Cursor Agent CLI reads instruction files automatically:

| File | Scope |
|------|-------|
| `.cursor/rules/*.md` | Project-level rules (glob-scoped via frontmatter) |
| `.cursor/rules/*.mdc` | Project-level rules (MDC format) |
| `CLAUDE.md` | Repository root — **also read by Cursor Agent CLI** |
| `AGENTS.md` | Repository root — also read |

**Key behavior**: Cursor Agent CLI reads `CLAUDE.md` automatically (same as Claude Code). This means the Session Start / Recovery procedure in CLAUDE.md applies directly. No separate auto-load file generation is needed (unlike Codex which requires AGENTS.md, or Copilot which requires .github/copilot-instructions.md).

## Commands

### Interactive Mode

Cursor Agent CLI uses a TUI interface. Key interactions:

| Action | Method |
|--------|--------|
| Submit prompt | Type + Enter |
| Cancel operation | Escape / Ctrl-C |
| Exit | Ctrl-C (when idle) or type `/exit` |

### Session Management

| Flag / Command | Purpose | Claude Code equivalent |
|----------------|---------|----------------------|
| `--resume [chatId]` | Select a session to resume | `claude --resume` |
| `--continue` | Continue most recent session | `claude --continue` |
| `--model <model>` | Select model at launch | `claude --model` |
| `--print` / `-p` | Non-interactive mode (stdout) | `claude -p` |

**No `/clear`, `/compact`, `/model` slash commands** in the current version. Context reset requires Ctrl-C + restart or a new `agent` invocation.

## Model Selection

### At Launch

```bash
agent --model opus-4.6-thinking    # Claude 4.6 Opus (Thinking) — default
agent --model sonnet-4.6-thinking  # Claude 4.6 Sonnet (Thinking)
agent --model sonnet-4.6           # Claude 4.6 Sonnet
agent --model gpt-5.3-codex       # GPT-5.3 Codex
agent --model gemini-3.1-pro      # Gemini 3.1 Pro
```

### In-Session

No `/model` command for runtime model switching. Model is fixed at launch.

### Available Models

Wide model selection including Claude (Opus/Sonnet 4.5-4.6), GPT (5.1-5.3 Codex variants), Gemini (3-3.1), and Grok. Use `agent models` or `--list-models` to see current list.

## MCP Configuration

Cursor Agent CLI reads MCP server configuration from `.cursor/mcp.json` (project-level) and `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@anthropic/memory-mcp"]
    }
  }
}
```

### MCP Management

| Command | Purpose |
|---------|---------|
| `agent mcp list` | List configured MCP servers |
| `agent mcp list-tools <id>` | List tools for a server |
| `agent mcp enable <id>` | Approve an MCP server |
| `agent mcp disable <id>` | Disable an MCP server |
| `agent mcp login <id>` | Authenticate with OAuth MCP server |
| `--approve-mcps` | Auto-approve all MCP servers at startup |

### Key differences from Claude Code MCP:

| Aspect | Claude Code | Cursor Agent CLI |
|--------|------------|------------------|
| Config format | JSON (`.mcp.json`) | JSON (`.cursor/mcp.json`) |
| Config location | Project root | `.cursor/` directory (project or global) |
| Auto-approve | N/A | `--approve-mcps` flag |
| Add command | `claude mcp add` | Manual JSON edit or IDE |
| Server types | stdio, SSE | stdio, SSE |

## Memory / State Management

### CLAUDE.md (auto-loaded)

Cursor Agent CLI reads `CLAUDE.md` automatically from the repository root — identical behavior to Claude Code. The Session Start / Recovery procedure applies directly.

### Session Persistence

Sessions stored in `~/.cursor/chats/`. Resume with:
- `--continue` — Most recent session
- `--resume [chatId]` — Select specific session
- `agent ls` — List available sessions
- `agent resume` — Resume latest session

### No Memory MCP equivalent

Cursor Agent CLI does not have a built-in persistent memory system. For cross-session knowledge, rely on:
- CLAUDE.md (project-level instructions)
- File-based state (queue/tasks/*.yaml, queue/reports/*.yaml)
- MCP servers if configured (e.g., Memory MCP)

## tmux Interaction

### Interactive Mode (`agent`)

- Cursor Agent CLI uses a TUI (similar to Codex, potentially alt-screen)
- send-keys compatibility needs testing — TUI may interfere
- capture-pane may be affected by alt-screen

### Non-Interactive Mode (`agent -p`)

- `--print` / `-p` — Runs headless, outputs to stdout
- `--output-format text|json|stream-json` for structured output
- `--trust` — Trust workspace without prompting (needed for headless)
- Ideal for tmux automation (no TUI interference)

### send-keys Compatibility

| Mode | send-keys | capture-pane | Notes |
|------|-----------|-------------|-------|
| TUI (default) | Risky (alt-screen possible) | Risky | Needs testing |
| Print mode (`-p`) | N/A (non-interactive) | stdout capture | Best for automation |

### Nudge Mechanism

For TUI mode:
- inbox_watcher.sh sends nudge text (e.g., `inbox3`) via tmux send-keys
- After receiving a nudge, the agent reads `queue/inbox/<agent>.yaml` and processes unread messages

For print mode:
- Each task is a separate `agent -p` invocation
- No nudge needed — task content is passed as argument

## Compaction Recovery

Cursor Agent CLI does not have `/clear` or `/compact` commands.

### Context Reset Options

1. **Ctrl-C + restart**: Kill the TUI session and launch a new `agent` instance
2. **New invocation**: Start a fresh `agent` session (old session can be resumed later)
3. **Print mode**: Each `agent -p` call is inherently a fresh context

### Shogun System Recovery (Cursor Ashigaru)

```
Step 1: CLAUDE.md is auto-loaded (contains recovery procedure)
Step 2: tmux display-message -t "$TMUX_PANE" -p '#{@agent_id}' → identify self
Step 3: Read queue/tasks/ashigaru{N}.yaml → determine current task
Step 4: If task has "target_path:" → read that file
Step 5: Resume work based on task status
```

**Note**: Like Codex, Cursor Agent CLI has no built-in Memory MCP. Recovery relies on CLAUDE.md + YAML files. However, Memory MCP can be configured via `.cursor/mcp.json`.

## Limitations (vs Claude Code)

| Feature | Claude Code | Cursor Agent CLI | Impact |
|---------|------------|------------------|--------|
| Memory MCP | Built-in | Not built-in (configurable) | Recovery relies on CLAUDE.md + files |
| Task tool (subagents) | Yes | No | Cannot spawn sub-agents |
| `/clear` context reset | Yes | No (Ctrl-C + restart) | Harder to reset mid-session |
| `/compact` compaction | Auto | No | No manual context compression |
| `/model` runtime switch | Yes (send-keys) | No | Model fixed at launch |
| Dynamic model switch | Full (via send-keys) | None in-session | Limited in automated mode |
| Prompt caching | 90% discount | Unknown (Cursor pricing) | Cost model differs |
| Cost model | API token-based | Subscription (requests/month) | Subscription limits apply |
| Alt-screen in tmux | No (terminal-native) | Possible (TUI) | tmux integration risk |
| Sandbox | None built-in | OS-level (optional) | `--sandbox enabled` available |
| Structured output | Text only | `json` / `stream-json` in print mode | Better for parsing |
| SemanticSearch | No | Yes | Cursor advantage for code understanding |
| Web search | WebSearch + WebFetch | WebFetch only | No dedicated web search tool |
| Multi-model selection | Claude models only | Claude + GPT + Gemini + Grok | Wider model access |

## Configuration Files Summary

| File | Location | Purpose |
|------|----------|---------|
| `cli-config.json` | `~/.cursor/` | Main CLI configuration (model, permissions, auth) |
| `mcp.json` | `~/.cursor/` or `.cursor/` | MCP server definitions |
| `CLAUDE.md` | Repo root | Project instructions (auto-loaded) |
| `.cursor/rules/*.md` | Project `.cursor/` | Project-level rules |
| `chats/` | `~/.cursor/chats/` | Session history |

## Command Line Reference

| Flag | Short | Purpose |
|------|-------|---------|
| `--model <model>` | | Override default model |
| `--yolo` / `--force` | `-f` | Auto-approve all tool calls |
| `--print` | `-p` | Non-interactive mode (stdout output) |
| `--output-format <fmt>` | | Output format: text, json, stream-json |
| `--trust` | | Trust workspace without prompting |
| `--workspace <path>` | | Set working directory |
| `--continue` | | Resume most recent session |
| `--resume [chatId]` | | Resume specific session |
| `--mode <mode>` | | Execution mode: plan, ask |
| `--plan` | | Start in plan mode (read-only) |
| `--cloud` | `-c` | Start in cloud mode |
| `--sandbox <mode>` | | Enable/disable sandbox |
| `--approve-mcps` | | Auto-approve all MCP servers |
| `--api-key <key>` | | API key for authentication |
| `--list-models` | | List available models and exit |
| `--stream-partial-output` | | Stream partial deltas (with -p + stream-json) |

## Subcommands

| Subcommand | Purpose |
|------------|---------|
| `agent login` | Authenticate with Cursor |
| `agent logout` | Clear authentication |
| `agent status` / `whoami` | View auth status |
| `agent models` | List available models |
| `agent about` | Display version, system, and account info |
| `agent update` | Update to latest version |
| `agent mcp` | Manage MCP servers |
| `agent ls` | List chat sessions |
| `agent resume` | Resume latest session |
| `agent create-chat` | Create new empty chat |
| `agent generate-rule` / `rule` | Generate a Cursor rule |

---

*Sources: `agent --help`, `agent about`, `agent models`, [Cursor CLI Documentation](https://docs.cursor.com/cli)*

# RepoFix

**A small, understandable AI coding agent for your terminal.**

RepoFix distills the core ideas behind modern coding agents into roughly 2,800 lines of Python. It can inspect a repository, edit files, run commands and tests, manage longer tasks, and work with any OpenAI-compatible LLM API.

The codebase is intentionally compact, making RepoFix useful both as a working CLI and as a reference implementation for learning how tool-using coding agents are built.

## Features

- **Provider-agnostic** — works with OpenAI-compatible APIs such as OpenAI, DeepSeek, local Ollama models, and more
- **Built-in coding tools** — read, search, create, edit, and run shell commands
- **Permission controls** — read-only tools run immediately; mutating tools require approval
- **Plan mode** — lets the agent investigate and propose a plan without changing anything
- **Undo checkpoints** — restores the most recent file change with `/undo`
- **Long-context management** — automatically trims and summarizes older context
- **Parallel tool execution** — runs independent tool calls concurrently
- **Sub-agents** — delegates complex work to an isolated agent context
- **Session persistence** — saves and resumes conversations across terminal sessions
- **Hooks** — runs custom commands before or after tool calls
- **MCP support** — connects external tools through stdio-based Model Context Protocol servers
- **Offline demo** — demonstrates the full agent loop without an API key

## Requirements

- Python 3.10 or newer
- An API key for an OpenAI-compatible model, unless you use the offline demo or a local server

## Installation

Clone the repository and install it in editable mode:

```bash
git clone https://github.com/thisXuan/RepoFix.git
cd RepoFix
python -m pip install -e .
```

Verify the renamed CLI or run the package directly:

```bash
repofix --version
python -m repofix --version
```

For development, install the test and lint dependencies:

```bash
python -m pip install -e ".[dev]"
```

To use providers that are not OpenAI-compatible, install the optional LiteLLM backend:

```bash
python -m pip install -e ".[litellm]"
```

## Quick Start

Set an API key, open a project directory, and start RepoFix:

```bash
export OPENAI_API_KEY="sk-..."
cd path/to/your/project
repofix
```

Then ask it to work on the project in natural language:

```text
You > Find the cause of the failing tests and fix it.
```

RepoFix reads the relevant files, asks for permission before making changes or running commands, and reports the result when the task is complete.

You can also run a single non-interactive prompt:

```bash
repofix -p "Explain how authentication works in this repository"
```

Mutating tools are refused in one-shot mode unless you explicitly allow them:

```bash
repofix --yes -p "Fix the failing tests and verify the changes"
```

## Offline Demo

Run a deterministic demo of the complete agent and tool loop without configuring an API key:

```bash
repofix --demo
```

The demo creates a temporary workspace, writes a Fibonacci implementation and test, runs pytest, and prints the final result.

There is also a more detailed plan-mode and hooks demo:

```bash
python examples/plan_hooks_demo.py
```

## Model Configuration

RepoFix reads configuration from environment variables and automatically loads a `.env` file from the current directory or one of its parents.

### OpenAI

```bash
export OPENAI_API_KEY="sk-..."
export REPOFIX_MODEL="gpt-5.5"
repofix
```

### OpenAI-compatible APIs

Set the API base URL and model name for providers such as DeepSeek or a self-hosted server:

```bash
export OPENAI_API_KEY="your-api-key"
export OPENAI_BASE_URL="https://api.deepseek.com"
export REPOFIX_MODEL="deepseek-chat"
repofix
```

### Ollama

Start Ollama locally, then point RepoFix at its OpenAI-compatible endpoint:

```bash
export OPENAI_API_KEY="ollama"
export OPENAI_BASE_URL="http://localhost:11434/v1"
export REPOFIX_MODEL="qwen2.5-coder"
repofix
```

### LiteLLM

For providers such as AWS Bedrock or Google Vertex AI, install the `litellm` extra and enable the LiteLLM backend:

```bash
export REPOFIX_PROVIDER="litellm"
export REPOFIX_MODEL="anthropic/claude-3-haiku"
repofix
```

Provider-specific credentials are still configured using the environment variables expected by LiteLLM.

### Environment Variables

| Variable | Default | Description |
| --- | --- | --- |
| `REPOFIX_MODEL` | `gpt-5.5` | Model name sent to the provider |
| `REPOFIX_API_KEY` | — | RepoFix-specific API key override |
| `OPENAI_API_KEY` | — | API key for OpenAI-compatible providers |
| `DEEPSEEK_API_KEY` | — | Fallback API key for DeepSeek |
| `REPOFIX_BASE_URL` | — | RepoFix-specific API base URL |
| `OPENAI_BASE_URL` | — | OpenAI-compatible API base URL |
| `REPOFIX_PROVIDER` | `openai` | Set to `litellm` to use the LiteLLM backend |
| `REPOFIX_MAX_TOKENS` | `4096` | Maximum completion tokens per model response |
| `REPOFIX_MAX_CONTEXT` | `128000` | Context size used by automatic compression |
| `REPOFIX_TEMPERATURE` | `0` | Sampling temperature |

Command-line options override environment variables:

```text
repofix [-m MODEL] [--base-url URL] [--api-key KEY]
        [-p PROMPT] [--yes] [--demo] [-r SESSION_ID]
```

### User Data

RepoFix stores local state under `~/.repofix/`, with command history in `~/.repofix_history`.

| Path | Purpose |
| --- | --- |
| `~/.repofix/permissions.json` | Persisted tool approvals |
| `~/.repofix/checkpoints.json` | File checkpoints used by `/undo` |
| `~/.repofix/sessions/` | Saved conversations |
| `~/.repofix/hooks.json` | Pre- and post-tool hooks |
| `~/.repofix/mcp.json` | MCP server configuration |
| `~/.repofix/pricing.json` | Optional model pricing overrides |
| `~/.repofix_history` | Interactive prompt history |

## Interactive Commands

| Command | Description |
| --- | --- |
| `/help` | Show the command reference |
| `/reset` | Clear the current conversation |
| `/model` | Show the current model |
| `/model <name>` | Switch models during a session |
| `/tokens` | Show token usage and estimated cost when pricing is known |
| `/compact` | Compress older conversation context |
| `/diff` | List files modified during the session |
| `/undo` | Revert the most recent `write_file` or `edit_file` change |
| `/plan` | Toggle read-only plan mode |
| `/save` | Save the current session |
| `/sessions` | List saved sessions |
| `/mcp` | Show MCP server status |
| `/mcp reconnect <name>` | Restart a disconnected MCP server |
| `quit` | Exit RepoFix |

Press **Enter** to submit a message and **Esc+Enter** to insert a newline.

## Safety and Permissions

RepoFix divides tools into two groups:

- Read-only tools such as file reading, globbing, searching, and task tracking run immediately.
- Mutating tools such as file writes, edits, shell commands, MCP calls, and sub-agents require approval.

In the interactive CLI, each request can be allowed once, always allowed for that tool, or denied. Persistent approvals are stored in `~/.repofix/permissions.json`.

Additional safeguards include:

- dangerous shell command pattern blocking
- exact, unique string matching for file edits
- automatic checkpoints before file writes
- read-only plan mode, even when `--yes` is active
- tool errors returned to the model instead of crashing the agent loop

`/undo` tracks changes made through `write_file` and `edit_file`. It does not revert side effects produced by shell commands.

## Hooks

Hooks let you validate, block, log, or react to tool calls. Create `~/.repofix/hooks.json`:

```json
{
  "PreToolUse": [
    {
      "matcher": "bash",
      "command": "python /path/to/check_command.py"
    }
  ],
  "PostToolUse": [
    {
      "matcher": "*",
      "command": "python /path/to/log_tool_call.py"
    }
  ]
}
```

Hook commands receive a JSON payload on standard input. A `PreToolUse` hook can block a call by exiting with status `2` and writing the reason to standard error. `PostToolUse` hooks observe completed calls but cannot block them.

Matchers accept an exact tool name, `*`, or an empty string for all tools.

## MCP Servers

RepoFix can load tools from stdio-based MCP servers. Configure them in `~/.repofix/mcp.json`:

```json
{
  "mcpServers": {
    "fs": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    }
  }
}
```

Remote tools are registered as `mcp__<server>__<tool>`. They pass through the same hooks and permission system as built-in tools. See [`examples/mcp_filesystem.md`](examples/mcp_filesystem.md) for a complete example.

## Using RepoFix as a Library

The agent can also be embedded in Python code:

```python
import os

from repofix import Agent, LLM

llm = LLM(
    model="gpt-5.5",
    api_key=os.environ["OPENAI_API_KEY"],
)
agent = Agent(llm=llm)

result = agent.chat("Summarize the structure of this repository")
print(result)
```

Pass a custom tool list, permission handler, hooks, or context limit to `Agent` when integrating it into another application.

## How It Works

At its core, RepoFix uses a small agent loop:

```text
user request
    ↓
LLM response
    ↓
tool calls? ── yes ──→ permission check → execute tools → return results to LLM
    │
    no
    ↓
final response
```

The main components are:

| Module | Responsibility |
| --- | --- |
| `repofix/agent.py` | Agent loop, tool execution, plan mode, and sub-agent wiring |
| `repofix/llm.py` | OpenAI-compatible streaming and LiteLLM support |
| `repofix/tools/` | Built-in repository and shell tools |
| `repofix/permissions.py` | Consent checks and persistent approvals |
| `repofix/context.py` | Multi-layer context compression |
| `repofix/checkpoints.py` | File mutation checkpoints and undo |
| `repofix/session.py` | Conversation save and resume |
| `repofix/hooks.py` | Pre- and post-tool shell hooks |
| `repofix/mcp.py` | MCP stdio client and remote tool registration |
| `repofix/cli.py` | Interactive terminal interface |

## Development

Run the test suite:

```bash
python -m pytest tests/ -v
```

Run the linter:

```bash
ruff check repofix tests
```

Build and verify the package:

```bash
python -m pip install build twine
python -m build
python -m twine check dist/*
```

The CI workflow tests Python 3.10 through 3.13 on Linux, macOS, and Windows.

## License

RepoFix is available under the [MIT License](LICENSE).

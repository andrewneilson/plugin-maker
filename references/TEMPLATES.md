# Plugin Templates

Copy-paste templates for creating plugin components.

## plugin.json Manifest

### Minimal (required fields only)

```json
{
  "name": "PLUGIN_NAME",
  "description": "PLUGIN_DESCRIPTION"
}
```

### Full (with optional fields)

```json
{
  "name": "PLUGIN_NAME",
  "version": "1.0.0",
  "description": "PLUGIN_DESCRIPTION",
  "author": {
    "name": "YOUR_NAME",
    "email": "YOUR_EMAIL"
  }
}
```

Replace:
- `PLUGIN_NAME`: lowercase with hyphens, matches directory name
- `PLUGIN_DESCRIPTION`: brief description of what the plugin does
- `YOUR_NAME`: your name (optional)
- `YOUR_EMAIL`: your email address (optional)

## README.md

```markdown
# Plugin Name

Brief description of what this plugin does.

## Commands

- `/plugin-name:command-name [args]` - Description of what the command does

## Agents

- `agent-name` - When this agent is used

## Development

```bash
# Test the plugin
claude --plugin-dir ./plugin-name
```

## Installation

```bash
cp -r ./plugin-name ~/.claude/plugins/
```

## Requirements

List any dependencies here.
```

## Command Frontmatter

### Basic Command

```yaml
---
description: What this command does
---

# Command Name

Instructions for Claude when this command is invoked.
```

### Command with Arguments

```yaml
---
description: What this command does
argument-hint: [required-arg] [optional-arg]
---

# Command Name

Handle the user's input from $ARGUMENTS.

If no arguments provided, ask for required information.
```

### Command with Tool Restrictions

```yaml
---
description: What this command does
argument-hint: [args]
allowed-tools: ["Read", "Write", "Bash", "Glob", "Grep"]
---

# Command Name

Instructions limited to the specified tools.
```

### Read-Only Command

```yaml
---
description: Analyze without making changes
allowed-tools: ["Read", "Grep", "Glob"]
---

# Command Name

Read-only analysis instructions.
```

## Agent Frontmatter

### Basic Agent

```yaml
---
name: agent-name
description: Use this agent when TRIGGER_SCENARIO.
model: sonnet
tools: ["Read", "Grep", "Glob"]
---

You are an expert at DOMAIN.

## Your Role

1. First responsibility
2. Second responsibility

## Output Format

How to structure your response.
```

### Agent with Examples (Recommended)

```yaml
---
name: security-analyzer
description: Use this agent for security analysis including vulnerability detection and authentication review. Examples: <example>Context: User asks about security\nuser: "Check auth.js for vulnerabilities"\nassistant: "I'll use security-analyzer for security review"</example> <example>Context: User mentions security\nuser: "Are there SQL injection risks?"\nassistant: "I'll analyze with security-analyzer"</example>
model: sonnet
color: red
tools: ["Read", "Grep", "Glob"]
---

You are an expert security analyst specializing in code security.

## Your Role

Analyze code for security vulnerabilities including:
1. Authentication and authorization issues
2. Injection vulnerabilities (SQL, XSS, command)
3. Cryptography misuse
4. Sensitive data exposure

## Output Format

### Critical Issues
- Issue with severity
- Location (file:line)
- Remediation steps

### Recommendations
- Security improvements
```

### Agent with Commentary

```yaml
---
name: refactoring-specialist
description: Use this agent when refactoring code or improving code structure. Examples: <example>Context: User wants better code\nuser: "Refactor this component"\nassistant: "I'll use refactoring-specialist to restructure"<commentary>Agent handles multi-file refactoring with consistency</commentary></example>
model: sonnet
color: green
tools: ["Read", "Grep", "Glob", "Edit"]
---

You are an expert in code refactoring and software design.

Improve code quality through strategic refactoring while preserving behavior.
```

### Lightweight Agent (Haiku)

```yaml
---
name: quick-checker
description: Fast validation for simple checks. Use for quick lookups and validations.
model: haiku
tools: ["Read", "Grep"]
---

Perform quick validation tasks.

Return results concisely.
```

## Skill SKILL.md

```yaml
---
name: skill-name
description: What this skill does. Use when user mentions TRIGGER_TERMS. Also use for RELATED_SCENARIOS.
---

# Skill Name

Brief introduction to the skill.

## Quick Start

Most common use case:

1. Step one
2. Step two

## Instructions

Detailed guidance for using this skill.

## Examples

Concrete, copy-paste ready examples.
```

## hooks.json

### Prompt-Based Hook (Recommended)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate file write safety. Check: system paths, credentials, path traversal. Return 'approve' or 'deny'.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Stop Hook for Completeness

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify task completion: tests run, build succeeded, questions answered. Return 'approve' to stop or 'block' with reason.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Command Hook (PreToolUse)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/pretooluse.py",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### All Hook Events

```json
{
  "hooks": {
    "PreToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/pretooluse.py",
        "timeout": 10
      }]
    }],
    "PostToolUse": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Analyze tool result for issues. Provide feedback if problems detected.",
        "timeout": 20
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Verify task completion. Return 'approve' or 'block' with reason.",
        "timeout": 30
      }]
    }],
    "SubagentStop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Verify subagent completed its task.",
        "timeout": 20
      }]
    }],
    "UserPromptSubmit": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Check if prompt requires security guidance. If yes, provide warnings.",
        "timeout": 20
      }]
    }],
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/load-context.sh",
        "timeout": 10
      }]
    }],
    "SessionEnd": [{
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/cleanup.sh",
        "timeout": 10
      }]
    }],
    "PreCompact": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Summarize key decisions and context to preserve during compaction.",
        "timeout": 20
      }]
    }],
    "Notification": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/notification.py",
        "timeout": 5
      }]
    }]
  }
}
```

### Hook with Matcher

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/check-bash.py",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

## Hook Handler Script (Python)

```python
#!/usr/bin/env python3
"""Hook handler template."""
import json
import sys

def main():
    # Read input from stdin
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    tool_input = input_data.get("tool_input", {})

    # Your logic here
    # Example: log, validate, modify, or block

    # Return response
    # Empty object allows tool to proceed
    # {"decision": "block", "reason": "..."} blocks the tool
    print(json.dumps({}))

if __name__ == "__main__":
    main()
```

### Hook Handler That Blocks

```python
#!/usr/bin/env python3
"""Hook handler that can block dangerous operations."""
import json
import sys

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    tool_input = input_data.get("tool_input", {})

    # Example: block rm -rf commands
    if tool_name == "Bash":
        command = tool_input.get("command", "")
        if "rm -rf" in command and "/" in command:
            print(json.dumps({
                "decision": "block",
                "reason": "Blocking potentially dangerous rm -rf command"
            }))
            return

    # Allow by default
    print(json.dumps({}))

if __name__ == "__main__":
    main()
```

## .lsp.json

### Basic LSP Configuration

```json
{
  "lspServers": {
    "server-name": {
      "command": "language-server-binary",
      "args": ["--stdio"],
      "filetypes": ["ext1", "ext2"]
    }
  }
}
```

### LSP with Environment Variables

```json
{
  "lspServers": {
    "server-name": {
      "command": "language-server-binary",
      "args": ["--stdio"],
      "filetypes": ["ext"],
      "env": {
        "MY_VAR": "value"
      }
    }
  }
}
```

## .mcp.json

### Basic MCP Configuration

```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"]
    }
  }
}
```

### MCP with Environment Variables

```json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
      "env": {
        "NODE_ENV": "production",
        "MY_VAR": "${MY_VAR}"
      }
    }
  }
}
```

### Python MCP Server

```json
{
  "mcpServers": {
    "server-name": {
      "command": "python3",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/server.py"]
    }
  }
}
```

### SSE MCP Server (Cloud)

```json
{
  "mcpServers": {
    "asana": {
      "type": "sse",
      "url": "https://mcp.asana.com/sse"
    }
  }
}
```

### HTTP MCP Server (REST API)

```json
{
  "mcpServers": {
    "api-service": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

### WebSocket MCP Server

```json
{
  "mcpServers": {
    "realtime": {
      "type": "ws",
      "url": "wss://realtime.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${WS_TOKEN}"
      }
    }
  }
}
```

## Directory Setup Commands

### Create Minimal Plugin (for development)

```bash
PLUGIN_NAME="my-plugin"
mkdir -p ./$PLUGIN_NAME/.claude-plugin
mkdir -p ./$PLUGIN_NAME/commands
```

### Create Full Plugin (for development)

```bash
PLUGIN_NAME="my-plugin"
mkdir -p ./$PLUGIN_NAME/.claude-plugin
mkdir -p ./$PLUGIN_NAME/commands
mkdir -p ./$PLUGIN_NAME/agents
mkdir -p ./$PLUGIN_NAME/skills/$PLUGIN_NAME
mkdir -p ./$PLUGIN_NAME/hooks
```

### Test Plugin During Development

```bash
claude --plugin-dir ./my-plugin
```

### Install Plugin

```bash
cp -r ./my-plugin ~/.claude/plugins/
```

### Create Project Plugin (committed to git)

```bash
PLUGIN_NAME="my-plugin"
mkdir -p .claude/plugins/$PLUGIN_NAME/.claude-plugin
mkdir -p .claude/plugins/$PLUGIN_NAME/commands
```

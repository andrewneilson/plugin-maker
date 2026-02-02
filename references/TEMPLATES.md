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

### Agent with Examples

```yaml
---
name: agent-name
description: Use this agent when analyzing code for issues. Examples: <example>Context: User wants security review\nuser: "Check this for vulnerabilities"\nassistant: "I'll use agent-name for analysis"</example>
model: sonnet
color: yellow
tools: ["Read", "Grep", "Glob", "Bash"]
---

You are an expert at DOMAIN.

## Core Mission

Detailed description of what this agent does.

## Process

1. Step one
2. Step two
3. Step three

## Output Format

### Section 1
- Item details

### Section 2
- Item details
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
            "prompt": "Validate file write safety. Check: path traversal, credentials, sensitive files. Return 'approve' or 'deny'.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Command Hook

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/validate.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### Multiple Hook Events

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate file write safety"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/log.sh",
            "timeout": 5
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify task completion: tests run, build succeeded"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/load-context.sh",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

### Hook with Matcher (Regex)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__.*__delete.*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate delete operation. Confirm this is intentional and safe."
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

## Plugin Settings Template

### Basic Settings File

```markdown
---
enabled: true
setting1: value1
setting2: value2
---

# Plugin Configuration

Additional context or documentation here.
```

### Settings with Multiple Types

```markdown
---
enabled: true
validation_level: strict
max_retries: 3
timeout_seconds: 30
allowed_extensions: [".js", ".ts", ".tsx"]
notification_level: info
---

# Project Settings

Settings are active for this project.
```

### Hook Settings Reader (Bash)

```bash
#!/bin/bash
set -euo pipefail

STATE_FILE=".claude/my-plugin.local.md"

# Quick exit if not configured
if [[ ! -f "$STATE_FILE" ]]; then
  exit 0
fi

# Parse frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$STATE_FILE")

# Read fields
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')
MODE=$(echo "$FRONTMATTER" | grep '^mode:' | sed 's/mode: *//' | sed 's/^"\(.*\)"$/\1/')

# Check enabled
if [[ "$ENABLED" != "true" ]]; then
  exit 0
fi

# Use settings
echo "Mode: $MODE"
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

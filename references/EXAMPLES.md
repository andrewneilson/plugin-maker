# Plugin Examples

Complete, working plugin examples demonstrating different configurations.

## Minimal Plugin (Command Only)

A simple plugin with just one slash command.

### Directory Structure

```
minimal-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── commands/
    └── greet.md
```

### Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "minimal-plugin",
  "description": "A minimal plugin demonstrating basic structure",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  }
}
```

**commands/greet.md**:
```yaml
---
description: Greet the user
argument-hint: [name]
---

# Greet Command

Greet the user by name.

If the user provides a name in $ARGUMENTS, use it. Otherwise, ask for their name.

Respond with a friendly greeting that includes:
1. A personalized hello
2. The current date
3. A relevant fun fact
```

**README.md**:
```markdown
# Minimal Plugin

A minimal Claude Code plugin demonstrating basic structure.

## Commands

- `/minimal-plugin:greet [name]` - Get a personalized greeting

## Development

```bash
claude --plugin-dir ./minimal-plugin
```

## Installation

```bash
cp -r ./minimal-plugin ~/.claude/plugins/
```
```

## Full-Featured Plugin (All Components)

A comprehensive plugin demonstrating commands, agents, skills, and hooks.

### Directory Structure

```
full-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
├── commands/
│   ├── analyze.md
│   └── help.md
├── agents/
│   └── code-analyzer.md
├── skills/
│   └── analysis-patterns/
│       └── SKILL.md
└── hooks/
    ├── hooks.json
    └── pre-analyze.py
```

### Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "full-plugin",
  "version": "1.0.0",
  "description": "Full-featured plugin with commands, agents, skills, and hooks",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  }
}
```

**commands/analyze.md**:
```yaml
---
description: Analyze code for patterns and issues
argument-hint: [file-or-directory]
allowed-tools: ["Read", "Grep", "Glob", "Task"]
---

# Analyze Command

Analyze code for patterns, potential issues, and improvement opportunities.

## Process

1. If $ARGUMENTS is provided, analyze that path
2. Otherwise, analyze the current directory
3. Use the code-analyzer agent for detailed analysis
4. Report findings with specific file and line references

## Output Format

Present findings as:
- **Critical**: Issues that must be fixed
- **Warnings**: Potential problems
- **Suggestions**: Improvement opportunities
```

**commands/help.md**:
```yaml
---
description: Get help with the full-plugin
allowed-tools: ["Read"]
---

# Full Plugin Help

Read the README.md file and present a summary of available commands and features.

Include usage examples for each command.
```

**agents/code-analyzer.md**:
```yaml
---
name: code-analyzer
description: Use this agent for deep code analysis including pattern detection, complexity assessment, and issue identification. Examples: <example>Context: User wants code reviewed\nuser: "Analyze src/ for issues"\nassistant: "I'll use the code-analyzer agent for deep analysis"</example>
model: sonnet
color: blue
tools: ["Read", "Grep", "Glob"]
---

You are an expert code analyst specializing in pattern detection and quality assessment.

## Your Role

Perform thorough code analysis including:
1. Pattern detection (design patterns, anti-patterns)
2. Complexity assessment (cyclomatic complexity, cognitive complexity)
3. Issue identification (bugs, security issues, performance problems)
4. Best practice evaluation

## Output Format

Return structured findings:

### Patterns Found
- Pattern name: location and description

### Issues
- Severity: description with file:line reference

### Recommendations
- Prioritized list of improvements
```

**skills/analysis-patterns/SKILL.md**:
```yaml
---
name: analysis-patterns
description: Detect and explain common code patterns and anti-patterns. Use when discussing design patterns, refactoring, or code quality.
---

# Analysis Patterns

Guide for detecting and explaining code patterns.

## Design Patterns

| Pattern | Detection Signals |
|---------|-------------------|
| Singleton | Private constructor, static instance |
| Factory | Create methods returning interface types |
| Observer | Subscribe/notify methods, event handlers |
| Strategy | Interface with multiple implementations |

## Anti-Patterns

| Anti-Pattern | Detection Signals |
|--------------|-------------------|
| God Object | Class with 500+ lines, many responsibilities |
| Spaghetti | Deep nesting, unclear control flow |
| Copy-Paste | Duplicate code blocks |
| Magic Numbers | Hardcoded values without constants |

## Usage

When analyzing code, identify both positive patterns and anti-patterns, providing specific recommendations for improvement.
```

**hooks/hooks.json**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/pre-analyze.py",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

**hooks/pre-analyze.py**:
```python
#!/usr/bin/env python3
"""Pre-analyze hook that logs file reads for analysis tracking."""
import json
import os
import sys
from datetime import datetime

def main():
    # Read hook input from stdin
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    tool_input = input_data.get("tool_input", {})

    # Log the file being read (for tracking purposes)
    if tool_name == "Read":
        file_path = tool_input.get("file_path", "")
        log_file = os.path.expanduser("~/.claude/analysis-log.txt")
        with open(log_file, "a") as f:
            f.write(f"{datetime.now().isoformat()} - Read: {file_path}\n")

    # Return empty response to allow the tool to proceed
    print(json.dumps({}))

if __name__ == "__main__":
    main()
```

## Plugin with MCP Integration

A plugin that provides an MCP server for additional capabilities.

### Directory Structure

```
mcp-plugin/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
├── README.md
├── commands/
│   └── mcp-status.md
└── mcp-server/
    ├── package.json
    └── index.js
```

### Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "mcp-plugin",
  "version": "1.0.0",
  "description": "Plugin demonstrating MCP server integration",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  }
}
```

**.mcp.json**:
```json
{
  "mcpServers": {
    "mcp-plugin-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```

**commands/mcp-status.md**:
```yaml
---
description: Check MCP server status
---

# MCP Status Command

Check the status of the plugin's MCP server.

1. Verify the MCP server is running
2. List available tools from the server
3. Report any connection issues
```

**mcp-server/package.json**:
```json
{
  "name": "mcp-plugin-server",
  "version": "1.0.0",
  "type": "module",
  "main": "index.js",
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  }
}
```

**README.md**:
```markdown
# MCP Plugin

Demonstrates MCP server integration with Claude Code plugins.

## Setup

1. Install dependencies: `cd mcp-server && npm install`

## Development

```bash
claude --plugin-dir ./mcp-plugin
```

## Installation

```bash
cp -r ./mcp-plugin ~/.claude/plugins/
```

## Commands

- `/mcp-plugin:mcp-status` - Check MCP server status

## MCP Tools

The bundled MCP server provides additional tools that appear in Claude's tool list.
```

## Plugin with Multiple Commands

A utility plugin with several related commands.

### Directory Structure

```
git-helpers/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── commands/
    ├── branch-cleanup.md
    ├── commit-stats.md
    ├── pr-summary.md
    └── help.md
```

**.claude-plugin/plugin.json**:
```json
{
  "name": "git-helpers",
  "version": "1.0.0",
  "description": "Git workflow helpers for branch management, commit analysis, and PR summaries",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  }
}
```

**commands/branch-cleanup.md**:
```yaml
---
description: Clean up merged and stale git branches
allowed-tools: ["Bash"]
---

# Branch Cleanup

Clean up local branches that have been merged or are stale.

## Process

1. List all local branches
2. Identify merged branches (excluding main/master)
3. Show stale branches (no commits in 30+ days)
4. Ask for confirmation before deleting
5. Delete approved branches

## Safety

- Never delete main, master, or develop
- Always confirm before deletion
- Show what would be deleted first
```

**commands/commit-stats.md**:
```yaml
---
description: Show commit statistics for the repository
argument-hint: [days]
allowed-tools: ["Bash"]
---

# Commit Stats

Show commit statistics for the repository.

If $ARGUMENTS specifies a number, show stats for that many days. Default: 30 days.

## Statistics to Show

1. Total commits
2. Commits by author
3. Most active days
4. Files changed most frequently
5. Average commit size
```

**commands/pr-summary.md**:
```yaml
---
description: Generate a summary of recent pull requests
argument-hint: [count]
allowed-tools: ["Bash"]
---

# PR Summary

Generate a summary of recent pull requests using `gh` CLI.

If $ARGUMENTS specifies a count, show that many PRs. Default: 10.

## Information to Include

1. PR title and number
2. Author
3. Status (open, merged, closed)
4. Files changed
5. Review status
```

## Plugin with Prompt-Based Hooks

A plugin demonstrating modern prompt-based hook validation.

### Directory Structure

```
security-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── hooks/
    └── hooks.json
```

### Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "security-plugin",
  "version": "1.0.0",
  "description": "Security validation using prompt-based hooks"
}
```

**hooks/hooks.json**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate file write safety. Check for: path traversal (../), sensitive files (.env, .key, credentials), system paths (/etc, /sys). Return 'approve' for safe operations or 'deny' with reason for unsafe operations.",
            "timeout": 30
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate bash command safety. Check for: rm -rf with system paths, curl/wget to unknown domains, sudo commands, chmod 777. Return 'approve' for safe commands or 'deny' with reason for dangerous commands."
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
            "prompt": "Verify task completion. Check: 1) Were tests run and passing? 2) Was code built successfully? 3) Were user questions answered? Return 'approve' to stop or 'block' with specific reason to continue."
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if prompt requires security guidance. If user mentions authentication, authorization, API keys, passwords, or security, return relevant security best practices as systemMessage."
          }
        ]
      }
    ]
  }
}
```

**README.md**:
```markdown
# Security Plugin

Automated security validation using AI-powered hooks.

## Features

- **File Write Validation**: Blocks path traversal and sensitive file access
- **Bash Command Safety**: Validates dangerous shell commands
- **Completion Verification**: Ensures tests run before stopping
- **Security Guidance**: Provides context-aware security advice

## Usage

The plugin automatically activates on:
- File writes and edits
- Bash commands
- Task completion
- User prompts mentioning security topics

No configuration required - hooks use AI reasoning to make decisions.
```

## Plugin with Settings Configuration

A plugin using per-project settings for customizable behavior.

### Directory Structure

```
configurable-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── hooks/
    ├── hooks.json
    └── validate.sh
```

### Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "configurable-plugin",
  "version": "1.0.0",
  "description": "Plugin with user-configurable validation levels"
}
```

**.claude/configurable-plugin.local.md** (user creates):
```markdown
---
enabled: true
validation_level: strict
max_file_size: 1000000
allowed_extensions: [".js", ".ts", ".tsx", ".json"]
enable_notifications: true
---

# Configurable Plugin Settings

This project uses strict validation mode.
All file writes are validated against security policies.
```

**hooks/hooks.json**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
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

**hooks/validate.sh**:
```bash
#!/bin/bash
set -euo pipefail

# Check for settings file
STATE_FILE=".claude/configurable-plugin.local.md"

if [[ ! -f "$STATE_FILE" ]]; then
  # No settings, use defaults
  exit 0
fi

# Parse frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$STATE_FILE")

# Check if enabled
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')
if [[ "$ENABLED" != "true" ]]; then
  exit 0
fi

# Read input
input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path // ""')

# Get validation level
LEVEL=$(echo "$FRONTMATTER" | grep '^validation_level:' | sed 's/validation_level: *//')

case "$LEVEL" in
  strict)
    # Strict: validate extension, size, path
    EXT="${file_path##*.}"
    ALLOWED=$(echo "$FRONTMATTER" | grep '^allowed_extensions:')

    if [[ ! "$ALLOWED" =~ ".$EXT" ]]; then
      echo '{"decision": "deny", "reason": "File extension not allowed in strict mode"}' >&2
      exit 2
    fi

    # Check for path traversal
    if [[ "$file_path" == *".."* ]]; then
      echo '{"decision": "deny", "reason": "Path traversal detected"}' >&2
      exit 2
    fi
    ;;

  standard)
    # Standard: basic checks
    if [[ "$file_path" == *".."* ]]; then
      echo '{"decision": "deny", "reason": "Path traversal detected"}' >&2
      exit 2
    fi
    ;;

  lenient)
    # Lenient: minimal checks
    ;;
esac

# Allow
exit 0
```

**README.md**:
```markdown
# Configurable Plugin

Plugin with per-project configuration for flexible validation.

## Configuration

Create `.claude/configurable-plugin.local.md` in your project:

\`\`\`markdown
---
enabled: true
validation_level: strict  # strict, standard, or lenient
max_file_size: 1000000
allowed_extensions: [".js", ".ts", ".tsx"]
enable_notifications: true
---

# Plugin Configuration
\`\`\`

Add to `.gitignore`:
\`\`\`
.claude/*.local.md
\`\`\`

### Validation Levels

- **strict**: Extension whitelist, path validation, size checks
- **standard**: Path validation only
- **lenient**: Minimal validation

### Changing Settings

After editing settings:
1. Save the file
2. Exit Claude Code
3. Restart: `claude` or `cc`

## Usage

Once configured, the plugin automatically validates file writes according to your settings.
```

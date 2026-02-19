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
  "description": "A minimal plugin demonstrating basic structure"
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

### Key Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "full-plugin",
  "version": "1.0.0",
  "description": "Full-featured plugin with commands, agents, skills, and hooks"
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

**agents/code-analyzer.md**:
```yaml
---
name: code-analyzer
description: Use this agent for deep code analysis. Examples: <example>Context: User wants code reviewed\nuser: "Analyze src/ for issues"\nassistant: "I'll use the code-analyzer agent for deep analysis"</example>
model: sonnet
color: blue
tools: ["Read", "Grep", "Glob"]
---

You are an expert code analyst specializing in pattern detection and quality assessment.

## Your Role

Perform thorough code analysis including:
1. Pattern detection (design patterns, anti-patterns)
2. Complexity assessment
3. Issue identification (bugs, security, performance)
4. Best practice evaluation

## Output Format

### Issues
- Severity: description with file:line reference

### Recommendations
- Prioritized list of improvements
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

For hook handler script examples, see [HOOKS.md](HOOKS.md).

## Plugin with MCP Integration

A plugin that bundles an MCP server for additional capabilities.

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

### Key Files

**.claude-plugin/plugin.json**:
```json
{
  "name": "mcp-plugin",
  "version": "1.0.0",
  "description": "Plugin demonstrating MCP server integration"
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

For MCP configuration details, see [MCP-INTEGRATION.md](MCP-INTEGRATION.md).

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
  "description": "Git workflow helpers for branch management, commit analysis, and PR summaries"
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

## Plugin with Prompt-Based Hooks

A plugin demonstrating prompt-based hook validation.

### Directory Structure

```
security-plugin/
├── .claude-plugin/
│   └── plugin.json
├── README.md
└── hooks/
    └── hooks.json
```

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
            "prompt": "Validate file write safety. Check for: path traversal, sensitive files (.env, .key), system paths. Return 'approve' or 'deny' with reason.",
            "timeout": 30
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
            "prompt": "Verify task completion: tests run, build succeeded, questions answered. Return 'approve' or 'block' with reason."
          }
        ]
      }
    ]
  }
}
```

For additional hook events and patterns, see [HOOKS.md](HOOKS.md).

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

**.claude-plugin/plugin.json**:
```json
{
  "name": "configurable-plugin",
  "version": "1.0.0",
  "description": "Plugin with user-configurable validation levels"
}
```

**.claude/configurable-plugin.local.md** (user creates in project):
```markdown
---
enabled: true
validation_level: strict
allowed_extensions: [".js", ".ts", ".tsx", ".json"]
---

# Configurable Plugin Settings

This project uses strict validation mode.
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

**hooks/validate.sh** (abbreviated):
```bash
#!/bin/bash
set -euo pipefail

STATE_FILE=".claude/configurable-plugin.local.md"
[[ ! -f "$STATE_FILE" ]] && exit 0

FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$STATE_FILE")
ENABLED=$(echo "$FRONTMATTER" | grep '^enabled:' | sed 's/enabled: *//')
[[ "$ENABLED" != "true" ]] && exit 0

input=$(cat)
file_path=$(echo "$input" | jq -r '.tool_input.file_path // ""')
LEVEL=$(echo "$FRONTMATTER" | grep '^validation_level:' | sed 's/validation_level: *//')

# Validate based on level (strict/standard/lenient)
if [[ "$file_path" == *".."* ]]; then
  echo '{"decision": "deny", "reason": "Path traversal detected"}' >&2
  exit 2
fi
```

For settings file format details, see [PLUGIN-SETTINGS.md](PLUGIN-SETTINGS.md).
For complete hook handler patterns, see [HOOKS.md](HOOKS.md).

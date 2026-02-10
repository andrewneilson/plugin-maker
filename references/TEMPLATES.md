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

### Important: Commands are Instructions FOR Claude

**Commands provide instructions FOR Claude, not TO the user.** Write from Claude's perspective about what Claude should do when the user runs this command.

❌ **Wrong** (instructions TO user):
```markdown
---
description: Run tests
---

# Test Command

Please run the following command to test your code:
```bash
npm test
```

Review the output and fix any failures.
```

✅ **Correct** (instructions FOR Claude):
```markdown
---
description: Run project tests and report results
---

# Test Command

When the user runs this command:

1. Execute the project's test suite using Bash
2. If using npm: run `npm test`
3. If using pytest: run `pytest`
4. Parse the test output
5. Report:
   - Number of tests passed/failed
   - Details of any failures
   - Suggestions for fixes if tests fail
```

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

### Command with Bash Tool Filtering

```yaml
---
description: Run git operations safely
allowed-tools: ["Bash(git:*)"]
---

# Git Command

You can only run git commands. Use Bash to execute git operations:
- git status
- git log
- git diff
- git branch

Do not run non-git commands.
```

### Command that Cannot be Programmatically Invoked

```yaml
---
description: Interactive command requiring user input
disable-model-invocation: true
---

# Interactive Setup

This command requires interactive user input and cannot be invoked automatically by Claude.

Steps:
1. Ask user for configuration details
2. Create configuration file
3. Confirm settings with user
```

### Command with Positional Arguments

```yaml
---
description: Process file with specified operation
argument-hint: <operation> <file-path>
---

# Process File Command

Arguments from $ARGUMENTS:
- $1: operation (validate|format|check)
- $2: file path

Steps:
1. Validate operation is one of: validate, format, check
2. Check file exists at $2
3. Perform the specified operation
4. Report results
```

## Command Patterns

### Pattern 1: Analysis Command (Read-Only)

```yaml
---
description: Analyze code structure and complexity
allowed-tools: ["Read", "Grep", "Glob"]
---

# Code Analysis

Analyze the project structure:

1. **Discover files**: Use Glob to find all source files
2. **Read files**: Use Read to examine code
3. **Analyze patterns**: Look for complexity, duplication
4. **Report findings**:
   - File organization
   - Code complexity metrics
   - Suggestions for improvement

Do not modify any files.
```

### Pattern 2: Interactive Command

```yaml
---
description: Interactive code refactoring with user approval
allowed-tools: ["Read", "Grep", "Write", "Edit"]
---

# Refactor Code

Interactive refactoring workflow:

1. **Analyze**: Read code to understand current structure
2. **Propose**: Show user planned refactoring changes
3. **Wait for approval**: Ask user "Proceed with these changes?"
4. **Execute**: Only if user approves, make changes
5. **Verify**: Show diff of changes made
6. **Test prompt**: Remind user to test changes
```

### Pattern 3: Build/Test Command

```yaml
---
description: Run tests and report results
allowed-tools: ["Bash", "Read"]
---

# Test Runner

Execute project tests:

1. **Detect test framework**:
   - Check for package.json → npm test
   - Check for pytest.ini → pytest
   - Check for Cargo.toml → cargo test

2. **Run tests**: Execute appropriate command

3. **Parse results**:
   - Count passed/failed tests
   - Extract failure details
   - Identify error patterns

4. **Report**:
   - ✅ "All 47 tests passed"
   - ❌ "3 of 50 tests failed: [details]"

5. **Suggest fixes** if failures detected
```

### Pattern 4: Code Generation Command

```yaml
---
description: Generate boilerplate code from template
allowed-tools: ["Read", "Write", "Create"]
---

# Generate Component

Generate code from templates:

1. **Get details from user**:
   - Component name
   - Component type
   - Optional features

2. **Read template**: Load template from ${CLAUDE_PLUGIN_ROOT}/templates/

3. **Customize**: Replace placeholders with user values

4. **Create file**: Write generated code

5. **Next steps**: Inform user of:
   - Files created
   - Import statements needed
   - How to use the component
```

### Pattern 5: Git Workflow Command

```yaml
---
description: Create feature branch and initial commit
allowed-tools: ["Bash(git:*)"]
---

# Start Feature

Git workflow automation:

1. **Branch name from user**: Ask for feature name

2. **Check status**:
   ```bash
   git status
   ```
   Ensure clean working directory

3. **Create branch**:
   ```bash
   git checkout -b feature/BRANCH_NAME
   ```

4. **Confirm**: Show user current branch

Only run git commands. Do not run other bash commands.
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

### Agent with Triggering Examples (Recommended)

```yaml
---
name: code-reviewer
description: This agent should be used when the user asks to "review my code", "check for code issues", "analyze code quality", or mentions code review, refactoring, or best practices
model: sonnet
color: blue
---

You are an expert code reviewer.

## Your Role

Analyze code for:
1. Correctness and logic errors
2. Code quality and maintainability
3. Security vulnerabilities
4. Performance issues

## Process

1. **Read the code**: Use Read/Grep to examine files
2. **Analyze thoroughly**: Check for issues systematically
3. **Provide feedback**: Clear, actionable suggestions
4. **Prioritize**: Critical issues first

## Output Format

### Critical Issues 🔴
- Security vulnerabilities
- Logic errors

### Improvements 🟡
- Code quality suggestions
- Performance optimizations

### Nitpicks 🔵
- Style improvements
- Minor suggestions

<example>
Context: User wants code review
user: "Can you review this authentication module?"
assistant: "I'll review your authentication code for security and best practices."
</example>

<example>
Context: Proactive review suggestion
user: "I just finished implementing the login feature"
assistant: "Great! Would you like me to review the login code for security issues?"
</example>
```

### Agent with Multiple Triggering Scenarios

```yaml
---
name: test-generator
description: This agent should be used when the user asks to "write tests", "create test cases", "add unit tests", "generate test coverage", or mentions testing, TDD, or test-driven development
model: sonnet
color: green
---

You are an expert test engineer specializing in comprehensive test coverage.

## Your Role

Generate thorough test suites including:
- Unit tests for individual functions
- Integration tests for component interactions
- Edge cases and boundary conditions
- Error handling scenarios

## Test Strategy

1. **Analyze code**: Understand functionality and dependencies
2. **Identify test cases**: Normal, edge, and error cases
3. **Write tests**: Clear, maintainable test code
4. **Verify coverage**: Ensure comprehensive coverage

<example>
Context: Direct test request
user: "Write tests for the UserService class"
assistant: "I'll create comprehensive unit tests for UserService including edge cases and error handling."
</example>

<example>
Context: Code completion trigger
user: "I finished implementing the payment processor"
assistant: "Would you like me to generate tests for the payment processor to ensure it handles all scenarios correctly?"
</example>

<example>
Context: Error fixing trigger
user: "The authentication is failing in production"
assistant: "Let me help debug this. First, do we have tests for the authentication flow? If not, I can generate them to reproduce the issue."
</example>
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

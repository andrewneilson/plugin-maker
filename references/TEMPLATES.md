# Plugin Templates

Copy-paste templates for creating plugin components. For hook, MCP, and settings templates, see their dedicated reference files.

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

Wrong (instructions TO user):
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

Correct (instructions FOR Claude):
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

Include `<example>` tags in the description for better agent discovery:

```yaml
---
name: code-reviewer
description: This agent should be used when the user asks to "review my code", "check for code issues", or mentions code review. Examples: <example>Context: User wants code review\nuser: "Check auth.js for vulnerabilities"\nassistant: "I'll use code-reviewer for security analysis"</example>
model: sonnet
color: blue
---

Agent instructions here.
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

## Additional Component Templates

For templates of these components, see their dedicated reference files:

- **Hook configuration**: See [HOOKS.md](HOOKS.md) for hooks.json format, handler scripts, and event patterns
- **MCP configuration**: See [MCP-INTEGRATION.md](MCP-INTEGRATION.md) for .mcp.json, server types, and tool naming
- **Plugin settings**: See [PLUGIN-SETTINGS.md](PLUGIN-SETTINGS.md) for settings files and reading configuration
- **Complete examples**: See [EXAMPLES.md](EXAMPLES.md) for full working plugin examples

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

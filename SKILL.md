---
name: plugin-maker
description: This skill should be used when the user asks to "create a plugin", "add a plugin", "make a plugin", "build a plugin", "create command", "add slash command", "create agent", "add hook", "create skill", "configure MCP server", "add MCP integration", "validate plugin", "debug plugin discovery", "use ${CLAUDE_PLUGIN_ROOT}", or mentions plugin structure, plugin.json manifests, hook events (PreToolUse, PostToolUse, Stop, SessionStart), MCP integration, agent triggering, or command development.
allowed-tools: Read Write Edit Bash Glob Grep
---

# Plugin Maker

Create discoverable Claude Code plugins that bundle commands, agents, skills, hooks, and MCP configurations.

## Core Principles

1. **Plugins bundle components**: A plugin is a container for related commands, agents, skills, hooks, and MCP configs
2. **Manifest-driven discovery**: The `.claude-plugin/plugin.json` file makes the plugin discoverable
3. **Convention over configuration**: Standard directory names enable automatic component discovery
4. **Plugins vs Skills**: Use plugins when bundling multiple component types; use standalone skills for single capabilities

## Plugin Structure

> **Warning**: Do not place `plugin.json` directly in the plugin directory. It must be inside the `.claude-plugin/` subdirectory for the plugin to be discovered.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Required: manifest file (MUST be here)
├── README.md                 # Recommended: documentation
├── commands/                 # Slash commands (*.md)
│   ├── my-command.md
│   └── help.md
├── agents/                   # Subagent definitions (*.md)
│   └── my-agent.md
├── skills/                   # Skills (subdirs with SKILL.md)
│   └── my-skill/
│       └── SKILL.md
├── hooks/                    # Hook configurations
│   ├── hooks.json
│   └── handler.py
├── .mcp.json                 # MCP server configuration
└── .lsp.json                 # LSP server configuration
```

## Quick Template: plugin.json

Every plugin requires `.claude-plugin/plugin.json`:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief description of what this plugin does"
}
```

Optional author field for attribution:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief description of what this plugin does",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  }
}
```

## Creating a Plugin

### Step 1: Create Directory Structure

```bash
mkdir -p ./my-plugin/.claude-plugin
mkdir -p ./my-plugin/commands
```

### Step 2: Create plugin.json

Create `.claude-plugin/plugin.json` with required fields:

```json
{
  "name": "my-plugin",
  "description": "What this plugin does"
}
```

### Step 3: Add Components

Add the components your plugin needs (see Component Quick Reference below).

### Step 4: Test with --plugin-dir

Use the `--plugin-dir` flag to test your plugin without installing it:

```bash
claude --plugin-dir ./my-plugin
```

This loads your plugin for the current session only, allowing rapid iteration.

### Step 5: Validate Structure

```bash
# Check manifest exists and is valid JSON
cat ./my-plugin/.claude-plugin/plugin.json | jq .

# List all components
ls -la ./my-plugin/
```

### Step 6: Test Discovery

Ask Claude to use your plugin's commands or agents to verify discovery works.

## Component Quick Reference

### Commands (`commands/*.md`)

Slash commands users invoke explicitly. Plugin commands are namespaced as `/plugin-name:command-name`.

```yaml
---
description: What this command does
argument-hint: [optional-args]
allowed-tools: ["Read", "Write", "Bash"]
---

# Command Name

Instructions for Claude when this command is invoked.

The user's input is available as $ARGUMENTS.
```

**Fields**:
| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | Shown in `/help` and command completion |
| `argument-hint` | No | Hint text for arguments (e.g., `[file-path]`) |
| `allowed-tools` | No | Restrict available tools (JSON array) |
| `disable-model-invocation` | No | If true, only explicit `/command` triggers it |

**Advanced Features**:
- **Bash execution**: Use `!git status` syntax to run commands inline
- **File references**: Use `@${CLAUDE_PLUGIN_ROOT}/config.json` to include file contents
- **Variables**: `$ARGUMENTS` for user input, `${CLAUDE_PLUGIN_ROOT}` for plugin paths
- See references/command-patterns.md for detailed patterns

### Agents (`agents/*.md`)

Specialized subagents for parallel or delegated work.

```yaml
---
name: my-agent
description: Use this agent when analyzing code for security issues. Examples: <example>Context: User asks about security\nuser: "Check this file for vulnerabilities"\nassistant: "I'll analyze with the security agent"</example>
model: sonnet
color: yellow
tools: ["Read", "Grep", "Glob"]
---

You are an expert at [specific task].

## Your Role

Detailed instructions for the agent...
```

**Fields**:
| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Agent identifier (lowercase, hyphens) |
| `description` | Yes | When to use (include Examples/commentary tags) |
| `model` | No | `inherit`, `sonnet`, `opus`, `haiku` |
| `color` | No | Visual indicator: yellow, blue, green, etc. |
| `tools` | No | Array of available tools |

**Agent Triggering Best Practices**:
- Include `Examples:` tag in description with `<example>` blocks
- Format: `Context: [situation]\nuser: "[quote]"\nassistant: "[response]"`
- Add `<commentary>` tags to explain triggering logic
- Use third-person: "Use this agent when..." not "Use me when..."
- See references/agent-patterns.md for detailed examples

### Skills (`skills/skill-name/SKILL.md`)

Model-invoked capabilities that Claude activates based on context. Plugin skills are namespaced as `/plugin-name:skill-name`.

```yaml
---
name: my-skill
description: What this skill does. Use when user mentions [trigger terms].
---

# Skill Name

Instructions...
```

### Hooks (`hooks/hooks.json`)

Event-driven automation that executes in response to Claude Code events.

**Hook Types**:
- **Prompt-based**: LLM-driven decisions for context-aware validation
- **Command-based**: Execute bash commands for deterministic checks

**Command hook example**:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/handler.py",
        "timeout": 10
      }]
    }]
  }
}
```

**Prompt-based hook example**:
```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Verify task completion: tests run, build succeeded. Return 'approve' to stop or 'block' with reason.",
        "timeout": 30
      }]
    }]
  }
}
```

**Available Events**:
| Event | Trigger |
|-------|---------|
| `PreToolUse` | Before any tool executes (can approve/deny) |
| `PostToolUse` | After a tool executes |
| `Stop` | When Claude wants to stop (validate completeness) |
| `SubagentStop` | When subagent wants to stop |
| `UserPromptSubmit` | When user submits a prompt |
| `SessionStart` | When Claude Code session begins |
| `SessionEnd` | When session ends |
| `PreCompact` | Before context compaction |
| `Notification` | When Claude sends notifications |

**Important**:
- Use `${CLAUDE_PLUGIN_ROOT}` for portable paths in hook commands
- Plugin hooks require wrapper format with `"hooks"` field (see references/hook-patterns.md)

### MCP Configuration (`.mcp.json`)

Model Context Protocol enables plugins to integrate with external services and APIs.

**Server Types**:
| Type | Use Case | Example |
|------|----------|---------|
| `stdio` | Local processes, custom servers | NPM-packaged MCP servers |
| `sse` | Cloud services with OAuth | Asana, GitHub hosted servers |
| `http` | REST APIs with token auth | Custom API backends |
| `ws` | Real-time communication | WebSocket-based services |

**stdio server (local process)**:
```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```

**SSE server (hosted service)**:
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

**HTTP server (REST API)**:
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

**Key points**:
- Use `${CLAUDE_PLUGIN_ROOT}` for portable paths
- MCP tools appear as: `mcp__plugin_<name>_<server>__<tool>`
- See references/mcp-patterns.md for detailed patterns

### LSP Configuration (`.lsp.json`)

Configure Language Server Protocol servers for enhanced editor support:

```json
{
  "lspServers": {
    "my-lsp": {
      "command": "my-language-server",
      "args": ["--stdio"],
      "filetypes": ["mylang"]
    }
  }
}
```

**Fields**:
| Field | Required | Description |
|-------|----------|-------------|
| `command` | Yes | The LSP server executable |
| `args` | No | Command-line arguments |
| `filetypes` | Yes | File extensions to activate for |

## Plugin Locations

| Location | Purpose |
|----------|---------|
| `--plugin-dir ./path` | Development/testing (primary workflow) |
| `~/.claude/plugins/` | Personal plugins (installed) |
| `.claude/plugins/` | Project plugins (committed to git) |

## When to Create a Plugin

### Standalone Skill vs Plugin Comparison

| Feature | Standalone Skill | Plugin |
|---------|------------------|--------|
| Simplest option | One SKILL.md file | Multiple files/directories |
| Commands | No | Yes |
| Multiple skills | No | Yes |
| Agents | No | Yes |
| Hooks | No | Yes |
| MCP servers | No | Yes |
| LSP servers | No | Yes |
| Namespacing | None | `/plugin-name:*` |
| Distribution | Copy file | Package directory |

**Create a plugin when**:
- You need multiple component types (commands + agents, skills + hooks, etc.)
- You want to bundle and distribute related functionality
- Your team needs shared tooling with consistent behavior

**Don't create a plugin when**:
- You only need a single skill (use standalone skill instead)
- You only need a single command (consider a skill or CLAUDE.md instruction)
- The functionality is project-specific and simple

## Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| Plugin directory | lowercase-hyphenated | `my-plugin` |
| plugin.json name | matches directory | `"name": "my-plugin"` |
| Commands directory | plural `commands/` | not `command/` |
| Agents directory | plural `agents/` | not `agent/` |
| Skills directory | plural `skills/` | not `skill/` |
| Hooks directory | singular `hooks/` | with `hooks.json` inside |

## Writing Style Guidelines

**For component descriptions** (commands, agents, skills):
- Use third-person: "Use this agent when..." not "Use me when..."
- Include specific trigger scenarios and keywords
- For agents, add `Examples:` tag with `<example>` blocks

**For component instructions** (body content):
- Use imperative/infinitive: "Review the code" not "You should review"
- Write directives TO Claude about what to do
- Avoid second-person phrases like "you should" or "you need to"

**Example correct style**:
```
Review this code for security vulnerabilities including:
- SQL injection
- XSS attacks

Provide specific line numbers and severity ratings.
```

**Example incorrect style**:
```
You should review this code. You'll receive a report with details.
```

## Quick Reference

**Create**:
```bash
mkdir -p ./my-plugin/.claude-plugin
```

**Test during development**:
```bash
claude --plugin-dir ./my-plugin
```

**Validate manifest**:
```bash
cat ./my-plugin/.claude-plugin/plugin.json | jq .
```

**Install plugin**:
```bash
cp -r ./my-plugin ~/.claude/plugins/
```

**List installed plugins**:
```bash
ls ~/.claude/plugins/*/
ls .claude/plugins/*/
```

## Additional Resources

- [Validation checklists](references/VALIDATION.md)
- [Complete plugin examples](references/EXAMPLES.md)
- [Common mistakes](references/MISTAKES.md)
- [Copy-paste templates](references/TEMPLATES.md)
- [Hook patterns and examples](references/hook-patterns.md)
- [MCP integration patterns](references/mcp-patterns.md)
- [Agent triggering patterns](references/agent-patterns.md)
- [Command advanced features](references/command-patterns.md)

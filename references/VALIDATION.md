# Plugin Validation

Complete validation checklists for ensuring plugins are properly structured and discoverable.

## Pre-Creation Checklist

**Verify BEFORE writing plugin content**:

- [ ] Plugin directory exists with correct name (lowercase, hyphens)
- [ ] `.claude-plugin/` subdirectory created
- [ ] `plugin.json` will have matching name field

## Manifest Validation

### Required Fields

| Field | Validation |
|-------|------------|
| `name` | Non-empty string, matches directory name, lowercase with hyphens |
| `description` | Non-empty string describing the plugin |

### Optional Fields

| Field | Type | Purpose |
|-------|------|---------|
| `version` | string | Semantic version (e.g., "1.0.0") |
| `author` | object | Attribution (with `name` and optional `email` fields) |

### JSON Validation Command

```bash
# Check JSON syntax
cat ./my-plugin/.claude-plugin/plugin.json | jq .

# Verify required fields exist
cat ./my-plugin/.claude-plugin/plugin.json | jq 'has("name") and has("description")'
```

## Component Validation

### Commands Checklist

- [ ] Files are in `commands/` directory (plural)
- [ ] Each file has `.md` extension
- [ ] YAML frontmatter has `description` field
- [ ] Frontmatter has opening `---` on line 1
- [ ] Frontmatter has closing `---` before content
- [ ] `allowed-tools` uses JSON array syntax if present
- [ ] `$ARGUMENTS` is documented if command accepts input

### Command Frontmatter Reference

**Required fields:**
```yaml
---
description: Brief description of what this command does
---
```

**Optional fields:**
```yaml
---
description: Command description
argument-hint: [required-arg] [optional-arg]  # Show expected arguments
allowed-tools: ["Read", "Write", "Bash"]      # Restrict tools available
disable-model-invocation: true                # Prevent programmatic invocation
model: sonnet                                 # Override model (sonnet, opus, haiku, inherit)
---
```

**Tool filtering:**
```yaml
---
allowed-tools: ["Bash(git:*)"]  # Only git commands
# or
allowed-tools: ["Bash(npm:*)"]  # Only npm commands
# or
allowed-tools: ["mcp__plugin_name_server__*"]  # All tools from MCP server
---
```

**Validate with yq:**
```bash
# Extract frontmatter
sed -n '/^---$/,/^---$/p' commands/my-command.md | yq .

# Check required fields
sed -n '/^---$/,/^---$/p' commands/my-command.md | yq '.description'
```

### Agents Checklist

- [ ] Files are in `agents/` directory (plural)
- [ ] Each file has `.md` extension
- [ ] YAML frontmatter has `name` field
- [ ] YAML frontmatter has `description` field
- [ ] `name` matches filename (without .md)
- [ ] Description includes trigger scenarios
- [ ] Description has Example/commentary tags for complex use cases
- [ ] `model` field is valid: `inherit`, `sonnet`, `opus`, or `haiku`
- [ ] `tools` field is array if present
- [ ] `color` field is valid if present

### Agent Frontmatter Reference

**Required fields:**
```yaml
---
name: agent-name
description: Use this agent when [trigger scenarios]
---
```

**Optional fields:**
```yaml
---
name: agent-name
description: Trigger scenarios and use cases
model: sonnet                     # inherit, sonnet, opus, haiku (default: inherit)
color: blue                       # blue, cyan, green, yellow, red, magenta
tools: ["Read", "Grep", "Glob"]   # Restrict tools (optional)
---
```

**Color meanings:**
- `blue`/`cyan`: Analysis, research, information gathering
- `green`: Success, validation, testing
- `yellow`: Caution, review, warning
- `red`: Critical, security, errors
- `magenta`: Creative, generation, transformation

**With triggering examples:**
```yaml
---
name: code-reviewer
description: This agent should be used when the user asks to "review code", "check for issues", or mentions code quality
---

<example>
Context: User wants code review
user: "Can you review this PR?"
assistant: "I'll use code-reviewer for comprehensive analysis"
</example>

<example>
Context: Proactive suggestion
user: "I finished the auth module"
assistant: "Great! Would you like me to review it for security issues?"
</example>
```

**Validate with sed/grep:**
```bash
# Extract frontmatter
sed -n '/^---$/,/^---$/p' agents/my-agent.md

# Check required fields
grep '^name:' agents/my-agent.md
grep '^description:' agents/my-agent.md

# Verify model value
grep '^model:' agents/my-agent.md | grep -E 'inherit|sonnet|opus|haiku'
```

### Skills Checklist

- [ ] Skills are in `skills/` directory (plural)
- [ ] Each skill has its own subdirectory
- [ ] Each skill directory contains `SKILL.md` file
- [ ] SKILL.md has valid YAML frontmatter
- [ ] `name` field matches subdirectory name
- [ ] `description` field is present and non-empty

### Hooks Checklist

- [ ] Hooks config is in `hooks/hooks.json`
- [ ] JSON is valid (no trailing commas)
- [ ] Plugin hooks wrapped in `{"hooks": {...}}` structure
- [ ] Event names are valid: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `SessionStart`, `SessionEnd`, `PreCompact`, `Notification`
- [ ] Hook type is valid: `"prompt"` or `"command"`
- [ ] Prompt-based hooks have `prompt` field
- [ ] Command hooks use `${CLAUDE_PLUGIN_ROOT}` for paths (not hardcoded)
- [ ] Handler scripts exist at referenced paths
- [ ] Handler scripts have execute permissions if needed
- [ ] Timeout values are reasonable (default: command 60s, prompt 30s)
- [ ] Matchers use valid patterns: exact match, regex, or `"*"`

### Hook Schema Validation

**Valid hook structure:**
```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolName|OtherTool",
        "hooks": [
          {
            "type": "prompt" | "command",
            "prompt": "..." (for prompt type),
            "command": "..." (for command type),
            "timeout": 30 (optional, in seconds)
          }
        ]
      }
    ]
  }
}
```

**Validate with jq:**
```bash
# Check structure
cat hooks/hooks.json | jq '.hooks'

# Verify event names
cat hooks/hooks.json | jq '.hooks | keys[]'

# Check for required fields
cat hooks/hooks.json | jq '.hooks.PreToolUse[]?.hooks[]? | has("type")'
```

### Plugin Settings Checklist

- [ ] Settings file uses format: `.claude/plugin-name.local.md`
- [ ] Plugin name in settings file matches plugin directory name
- [ ] Settings file has valid YAML frontmatter (between `---` markers)
- [ ] Hooks/commands that read settings handle missing file gracefully
- [ ] README documents settings file structure and location
- [ ] README documents restart requirement for settings changes
- [ ] `.gitignore` includes `.claude/*.local.md`
- [ ] Settings parsing validates values before use

### MCP Configuration Checklist

- [ ] Config is in `.mcp.json` at plugin root
- [ ] JSON is valid
- [ ] Commands use `${CLAUDE_PLUGIN_ROOT}` for paths
- [ ] Referenced server scripts exist
- [ ] Dependencies are documented in README

### LSP Configuration Checklist

- [ ] Config is in `.lsp.json` at plugin root
- [ ] JSON is valid
- [ ] `command` field specifies the LSP server executable
- [ ] `filetypes` array specifies which file extensions activate it
- [ ] LSP server is installed and available in PATH

## Post-Creation Validation Protocol

### Step 1: Verify directory structure

```bash
# Check plugin root
ls -la ./my-plugin/

# Check manifest
ls -la ./my-plugin/.claude-plugin/
```

### Step 2: Validate manifest JSON

```bash
cat ./my-plugin/.claude-plugin/plugin.json | jq .
```

You should see properly formatted JSON with no errors.

### Step 3: Verify component directories

```bash
# Check for correct plural names
ls ./my-plugin/ | grep -E '^(commands|agents|skills|hooks)$'
```

### Step 4: Test with --plugin-dir

```bash
# Test plugin without installing
claude --plugin-dir ./my-plugin
```

Then inside Claude:
1. Ask Claude about your plugin's functionality
2. Try using a command: `/my-plugin:my-command`
3. Check if agents appear in Task tool options

### Step 5: Debug if needed

```bash
# Run Claude with debug output
claude --debug --plugin-dir ./my-plugin
```

## Common Validation Errors

### "Plugin not found"

1. Check `.claude-plugin/plugin.json` exists
2. Verify JSON is valid (no trailing commas)
3. Confirm `name` field matches directory name

### "Command not available"

1. Verify file is in `commands/` (not `command/`)
2. Check frontmatter syntax
3. Ensure `description` field exists

### "Agent not triggering"

1. Verify file is in `agents/` (not `agent/`)
2. Check `description` includes trigger scenarios
3. Add Example tags for clarity

### "Hook not executing"

1. Verify `hooks/hooks.json` exists
2. Check JSON syntax (common: trailing commas)
3. Ensure `${CLAUDE_PLUGIN_ROOT}` is used for paths
4. Verify handler scripts have execute permissions

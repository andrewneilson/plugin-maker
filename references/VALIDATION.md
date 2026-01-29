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

### Agents Checklist

- [ ] Files are in `agents/` directory (plural)
- [ ] Each file has `.md` extension
- [ ] YAML frontmatter has `name` field
- [ ] YAML frontmatter has `description` field
- [ ] `name` matches filename (without .md)
- [ ] Description uses third-person: "Use this agent when..." not "Use me when..."
- [ ] Description includes specific trigger scenarios
- [ ] Description has `Examples:` tag with `<example>` blocks
- [ ] Examples include: Context, user quote, assistant response
- [ ] Examples use proper format: `<example>Context: ...\nuser: "..."\nassistant: "..."</example>`
- [ ] Optional `<commentary>` tags explain complex triggering logic
- [ ] `model` field is valid: `inherit`, `sonnet`, `opus`, or `haiku`
- [ ] `tools` field is array if present
- [ ] `color` field uses valid color if present

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
- [ ] Plugin hooks use wrapper format with `"hooks"` field
- [ ] Event names are valid: `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStop`, `UserPromptSubmit`, `SessionStart`, `SessionEnd`, `PreCompact`, `Notification`
- [ ] Hook type is either `"command"` or `"prompt"`
- [ ] Prompt-based hooks have reasonable timeout (20-30 seconds)
- [ ] Command hooks use `${CLAUDE_PLUGIN_ROOT}` for paths (not hardcoded)
- [ ] Handler scripts exist at referenced paths
- [ ] Handler scripts have execute permissions if needed
- [ ] Timeout values are reasonable (5-30 seconds)
- [ ] Matchers use valid tool names or wildcards

### MCP Configuration Checklist

- [ ] Config is in `.mcp.json` at plugin root (or inline in plugin.json)
- [ ] JSON is valid
- [ ] Server type specified if not stdio: `"sse"`, `"http"`, `"ws"`
- [ ] stdio servers use `${CLAUDE_PLUGIN_ROOT}` for portable paths
- [ ] SSE servers have valid `url` field
- [ ] HTTP servers have `url` and optional `headers`
- [ ] WebSocket servers have valid `url`
- [ ] Environment variables documented in README
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

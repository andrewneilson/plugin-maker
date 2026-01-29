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
| `author` | Object with `name` field (email optional) |

### Optional Fields

| Field | Type | Purpose |
|-------|------|---------|
| `version` | string | Semantic version (e.g., "1.0.0") |
| `author.email` | string | Contact email |

### JSON Validation Command

```bash
# Check JSON syntax
cat ~/.claude/plugins/my-plugin/.claude-plugin/plugin.json | jq .

# Verify required fields exist
cat ~/.claude/plugins/my-plugin/.claude-plugin/plugin.json | jq 'has("name") and has("description") and has("author")'
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
- [ ] Description includes trigger scenarios
- [ ] Description has Example/commentary tags for complex use cases
- [ ] `model` field is valid: `inherit`, `sonnet`, `opus`, or `haiku`
- [ ] `tools` field is array if present

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
- [ ] Event names are valid: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`
- [ ] Commands use `${CLAUDE_PLUGIN_ROOT}` for paths (not hardcoded)
- [ ] Handler scripts exist at referenced paths
- [ ] Handler scripts have execute permissions if needed
- [ ] Timeout values are reasonable (default 10-30 seconds)

### MCP Configuration Checklist

- [ ] Config is in `.mcp.json` at plugin root
- [ ] JSON is valid
- [ ] Commands use `${CLAUDE_PLUGIN_ROOT}` for paths
- [ ] Referenced server scripts exist
- [ ] Dependencies are documented in README

## Post-Creation Validation Protocol

### Step 1: Verify directory structure

```bash
# Check plugin root
ls -la ~/.claude/plugins/my-plugin/

# Check manifest
ls -la ~/.claude/plugins/my-plugin/.claude-plugin/
```

### Step 2: Validate manifest JSON

```bash
cat ~/.claude/plugins/my-plugin/.claude-plugin/plugin.json | jq .
```

You should see properly formatted JSON with no errors.

### Step 3: Verify component directories

```bash
# Check for correct plural names
ls ~/.claude/plugins/my-plugin/ | grep -E '^(commands|agents|skills|hooks)$'
```

### Step 4: Test discovery

1. Ask Claude about your plugin's functionality
2. Try using a command: `/my-command`
3. Check if agents appear in Task tool options

### Step 5: Debug if needed

```bash
# Run Claude with debug output
claude --debug
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

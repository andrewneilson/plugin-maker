# Plugin Manifest Reference

Complete guide to configuring `plugin.json` manifests for Claude Code plugins.

## Overview

The plugin manifest (`plugin.json`) defines plugin metadata, configuration, and component paths. It must be located at `.claude-plugin/plugin.json` within your plugin directory.

## Required Fields

### name

The unique identifier for your plugin. Must be kebab-case.

```json
{
  "name": "my-plugin"
}
```

**Requirements:**
- Use kebab-case format (lowercase with hyphens)
- No spaces or special characters except hyphens
- Must be unique across installed plugins
- Regex: `/^[a-z][a-z0-9]*(-[a-z0-9]+)*$/`

**Examples:**
- ✅ `code-review-assistant`
- ✅ `test-runner`
- ✅ `api-docs`
- ❌ `Code-Review` (uppercase)
- ❌ `test_runner` (underscore)
- ❌ `my plugin` (space)

## Recommended Metadata Fields

### version

Semantic version number following SemVer format.

```json
{
  "version": "1.0.0"
}
```

**Format:** `MAJOR.MINOR.PATCH`
- **MAJOR**: Breaking changes (incompatible API changes)
- **MINOR**: New features (backward-compatible)
- **PATCH**: Bug fixes (backward-compatible)

**Pre-release versions:**
```json
{
  "version": "1.0.0-alpha.1"    // Alpha release
}
```
```json
{
  "version": "2.0.0-beta.2"     // Beta release
}
```
```json
{
  "version": "1.5.0-rc.1"       // Release candidate
}
```

**Development versions:**
```json
{
  "version": "1.0.0-dev"        // Development build
}
```

**Best practices:**
- Start at `0.1.0` for initial development
- Use `1.0.0` for first stable release
- Increment MAJOR for breaking changes
- Increment MINOR for new features
- Increment PATCH for bug fixes
- Use pre-release suffixes for testing

### description

Brief explanation of plugin purpose (1-2 sentences).

```json
{
  "description": "Automated code review assistant that validates PR changes and suggests improvements"
}
```

**Best practices:**
- Keep under 160 characters
- Focus on what the plugin does
- Mention key features
- Use active voice

### author

Plugin author information.

**Full format:**
```json
{
  "author": {
    "name": "Jane Developer",
    "email": "jane@example.com",
    "url": "https://janedeveloper.com"
  }
}
```

**Minimal format:**
```json
{
  "author": {
    "name": "Jane Developer"
  }
}
```

**String shorthand:**
```json
{
  "author": "Jane Developer <jane@example.com>"
}
```

**Fields:**
- `name` (required): Author's name or organization
- `email` (optional): Contact email
- `url` (optional): Website or profile URL

### homepage

Plugin documentation or homepage URL.

```json
{
  "homepage": "https://docs.example.com/my-plugin"
}
```

**Use for:**
- Plugin documentation
- Project website
- Usage guides
- Tutorial links

### repository

Source code repository URL or object.

**String format:**
```json
{
  "repository": "https://github.com/username/my-plugin"
}
```

**Object format:**
```json
{
  "repository": {
    "type": "git",
    "url": "https://github.com/username/my-plugin.git"
  }
}
```

**Supported types:**
- `git`
- `svn`
- `hg`

### license

SPDX license identifier.

```json
{
  "license": "MIT"
}
```

**Common licenses:**
- `MIT` - Permissive, widely used
- `Apache-2.0` - Permissive with patent grant
- `GPL-3.0` - Copyleft
- `BSD-3-Clause` - Permissive
- `ISC` - Simplified permissive
- `UNLICENSED` - Proprietary, no license

### keywords

Array of keywords for plugin discovery and categorization.

```json
{
  "keywords": ["testing", "automation", "ci-cd", "quality"]
}
```

**Best practices:**
- Use 3-7 keywords
- Include technology names (e.g., "python", "react")
- Include use cases (e.g., "testing", "deployment")
- Use lowercase
- Be specific and relevant

## Component Path Customization

Override default component discovery paths. Custom paths supplement (not replace) defaults.

### commands

Custom path for command files.

**Single path:**
```json
{
  "commands": "./custom-commands"
}
```

**Multiple paths:**
```json
{
  "commands": ["./commands", "./specialized-commands"]
}
```

**Default:** `./commands/` directory

### agents

Custom path for agent files.

**Single path:**
```json
{
  "agents": "./custom-agents"
}
```

**Multiple paths:**
```json
{
  "agents": ["./agents", "./specialized-agents"]
}
```

**Default:** `./agents/` directory

### skills

Custom path for skill directories (not supported via manifest - skills must be in `./skills/`).

**Default:** `./skills/` directory with `SKILL.md` in each subdirectory

### hooks

Custom path for hooks configuration file.

```json
{
  "hooks": "./config/hooks.json"
}
```

**Default:** `./hooks/hooks.json`

### mcpServers

Inline MCP server configuration (alternative to `.mcp.json`).

```json
{
  "mcpServers": {
    "my-server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/server.js",
      "args": ["--port", "8080"]
    }
  }
}
```

**Default:** `./.mcp.json` at plugin root

**See:** [MCP-INTEGRATION.md](MCP-INTEGRATION.md) for detailed MCP configuration.

## Path Requirements

All custom paths must:
- Be relative to plugin root
- Start with `./`
- Not use absolute paths
- Point to existing files/directories
- Not use `../` (no parent directory traversal)

**Examples:**

✅ Valid paths:
```json
{
  "commands": "./custom-commands",
  "agents": ["./agents", "./specialized-agents"],
  "hooks": "./config/hooks.json"
}
```

❌ Invalid paths:
```json
{
  "commands": "/absolute/path",      // Absolute path
  "agents": "relative/without/dot",  // Missing ./
  "hooks": "../parent/hooks.json"    // Parent traversal
}
```

## Complete Example

Full-featured plugin.json with all recommended fields:

```json
{
  "name": "code-review-assistant",
  "version": "1.2.3",
  "description": "Automated code review assistant that validates PR changes, runs linters, and suggests improvements",
  "author": {
    "name": "Jane Developer",
    "email": "jane@example.com",
    "url": "https://janedeveloper.com"
  },
  "homepage": "https://docs.example.com/code-review-assistant",
  "repository": {
    "type": "git",
    "url": "https://github.com/username/code-review-assistant.git"
  },
  "license": "MIT",
  "keywords": [
    "code-review",
    "linting",
    "quality",
    "automation",
    "ci-cd"
  ],
  "commands": "./commands",
  "agents": ["./agents", "./specialized-reviewers"],
  "hooks": "./config/hooks.json",
  "mcpServers": {
    "github-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/github-server.js",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## Minimal Example

Simplest valid plugin.json:

```json
{
  "name": "my-plugin"
}
```

This is sufficient for plugin discovery. Other fields are optional but recommended.

## Component Discovery Behavior

### Default Discovery

Without custom paths, Claude Code auto-discovers:
- Commands: `./commands/*.md`
- Agents: `./agents/*.md`
- Skills: `./skills/*/SKILL.md`
- Hooks: `./hooks/hooks.json`
- MCP servers: `./.mcp.json`

### Custom Path Behavior

Custom paths **supplement** defaults:

```json
{
  "name": "my-plugin",
  "commands": "./extra-commands"
}
```

Claude Code will load commands from:
1. `./commands/` (default)
2. `./extra-commands/` (custom)

### Arrays for Multiple Paths

```json
{
  "agents": ["./agents", "./specialized", "./experimental"]
}
```

Claude Code will load agents from all three directories.

## Validation

### Schema Validation

The manifest must be valid JSON and conform to the schema:

```json
{
  "name": "string (required, kebab-case)",
  "version": "string (optional, semver)",
  "description": "string (optional)",
  "author": "string or object (optional)",
  "homepage": "string (optional)",
  "repository": "string or object (optional)",
  "license": "string (optional)",
  "keywords": ["string array (optional)"],
  "commands": "string or string array (optional)",
  "agents": "string or string array (optional)",
  "hooks": "string (optional)",
  "mcpServers": "object (optional)"
}
```

### Common Validation Errors

**Invalid name format:**
```json
{
  "name": "My_Plugin"  // ❌ Must be kebab-case
}
```

**Invalid version:**
```json
{
  "version": "1.0"  // ❌ Must be MAJOR.MINOR.PATCH
}
```

**Invalid paths:**
```json
{
  "commands": "../commands"  // ❌ No parent traversal
}
```

## Best Practices

### Metadata Completeness

**DO:**
- ✅ Include version, description, author
- ✅ Provide homepage for documentation
- ✅ Link to repository for source
- ✅ Specify license clearly
- ✅ Use relevant keywords

**DON'T:**
- ❌ Leave fields blank or with placeholder text
- ❌ Use outdated version numbers
- ❌ Omit author information
- ❌ Skip license field

### Versioning Strategy

**DO:**
- ✅ Start at 0.1.0 for development
- ✅ Use 1.0.0 for first stable release
- ✅ Follow semantic versioning strictly
- ✅ Document breaking changes in MAJOR updates
- ✅ Use pre-release suffixes for testing

**DON'T:**
- ❌ Skip versions (e.g., 1.0.0 → 2.0.0 without 1.x.x releases)
- ❌ Use arbitrary version numbers
- ❌ Forget to update version on release
- ❌ Break compatibility in MINOR updates

### Component Paths

**DO:**
- ✅ Rely on defaults when possible
- ✅ Use custom paths for logical organization
- ✅ Document custom paths in README
- ✅ Keep paths relative and portable

**DON'T:**
- ❌ Use custom paths unnecessarily
- ❌ Use absolute paths
- ❌ Create deep directory hierarchies
- ❌ Mix component types in single directory

### MCP Configuration

**DO:**
- ✅ Use `.mcp.json` for complex MCP setups
- ✅ Use inline `mcpServers` for simple single-server plugins
- ✅ Use ${CLAUDE_PLUGIN_ROOT} for portable paths
- ✅ Document required environment variables

**DON'T:**
- ❌ Duplicate MCP config (use either .mcp.json or inline, not both)
- ❌ Hardcode credentials in manifest
- ❌ Use absolute paths
- ❌ Forget to document authentication requirements

## Troubleshooting

### Plugin Not Discovered

**Check:**
- File is at `.claude-plugin/plugin.json` (not `plugin.json` at root)
- JSON is valid (use `jq . < plugin.json`)
- `name` field exists and is valid kebab-case
- Plugin directory is in correct location

### Components Not Loading

**Check:**
- Custom paths are correct and exist
- Paths start with `./`
- Files have correct extensions (.md for commands/agents)
- Skills have SKILL.md in subdirectories

### MCP Servers Not Starting

**Check:**
- MCP configuration is valid JSON
- Paths use ${CLAUDE_PLUGIN_ROOT}
- Required environment variables are set
- Server executables exist and are executable

## Quick Reference

### Minimal Manifest

```json
{
  "name": "my-plugin"
}
```

### Complete Manifest Template

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "What this plugin does",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "homepage": "https://docs.example.com",
  "repository": "https://github.com/you/plugin-name",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"]
}
```

### With Custom Paths

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "commands": ["./commands", "./extra-commands"],
  "agents": "./agents",
  "hooks": "./config/hooks.json"
}
```

### With Inline MCP

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "mcpServers": {
    "server-name": {
      "command": "${CLAUDE_PLUGIN_ROOT}/server.js",
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

# Common Mistakes

The most frequent issues that prevent plugins from working correctly.

## 1. Missing .claude-plugin Directory (CRITICAL)

The plugin won't be discovered without this directory.

**Wrong** - No .claude-plugin directory:
```
my-plugin/
├── plugin.json         # Wrong location!
└── commands/
    └── my-command.md
```

**Correct** - plugin.json inside .claude-plugin:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json     # Correct location
└── commands/
    └── my-command.md
```

## 2. Invalid plugin.json JSON

Trailing commas and syntax errors prevent plugin loading.

**Wrong** - Trailing comma:
```json
{
  "name": "my-plugin",
  "description": "My plugin",
  "author": {
    "name": "Me",
  }
}
```

**Correct** - Valid JSON (no trailing commas):
```json
{
  "name": "my-plugin",
  "description": "My plugin",
  "author": {
    "name": "Me"
  }
}
```

Validate with: `cat .claude-plugin/plugin.json | jq .`

## 3. Name Mismatch

The `name` field must match the plugin directory name.

**Wrong**: Directory is `my-plugin/` but plugin.json has `"name": "myPlugin"`

**Correct**: Directory is `my-plugin/` and plugin.json has `"name": "my-plugin"`

## 4. Wrong Directory Names (Singular vs Plural)

Component directories must use the correct plural forms.

**Wrong** - Singular names:
```
my-plugin/
├── command/              # Wrong: should be 'commands'
├── agent/                # Wrong: should be 'agents'
└── skill/                # Wrong: should be 'skills'
```

**Correct** - Plural names:
```
my-plugin/
├── commands/             # Correct: plural
├── agents/               # Correct: plural
└── skills/               # Correct: plural
```

## 5. Skill Not in Subdirectory

Skills must be in their own subdirectory within `skills/`.

**Wrong**: `skills/SKILL.md`

**Correct**: `skills/my-skill/SKILL.md`

## 6. Hooks Missing Handler Files

Hook commands reference scripts that don't exist.

**Wrong** - hooks.json references `handler.py` but the file doesn't exist:
```json
{
  "hooks": {
    "PreToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/handler.py"
      }]
    }]
  }
}
```

**Fix**: Create the handler file and ensure it's executable.

## 7. Agent Description Without Trigger Examples

Agent descriptions should include `<example>` tags for better discovery.

**Wrong** - Generic description:
```yaml
---
name: code-analyzer
description: Analyzes code
---
```

**Correct** - With trigger scenarios and examples:
```yaml
---
name: code-analyzer
description: Use this agent for deep code analysis. Examples: <example>Context: User wants security review\nuser: "Check auth.js for vulnerabilities"\nassistant: "I'll use code-analyzer for security analysis"</example>
---
```

## 8. Command Without $ARGUMENTS Handling

Commands with `argument-hint` should reference `$ARGUMENTS` in the body and document fallback behavior when no arguments are provided.

**Wrong** - Has `argument-hint: [name]` but body never mentions `$ARGUMENTS`.

**Correct**:
```yaml
---
description: Greet the user
argument-hint: [name]
---

# Greet

If the user provides a name in $ARGUMENTS, use it. Otherwise, ask for their name.
```

## 9. Hardcoded Paths

Hook commands and MCP server configurations must use `${CLAUDE_PLUGIN_ROOT}`.

**Wrong** - Hardcoded paths:
```json
"command": "python3 /Users/me/.claude/plugins/my-plugin/hooks/handler.py"
"command": "/Users/me/.claude/plugins/my-plugin/servers/server.js"
```

**Correct** - Use variable:
```json
"command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/handler.py"
"command": "${CLAUDE_PLUGIN_ROOT}/servers/server.js"
```

This applies to both hook handler paths and MCP server paths.

## 10. Tools Field Wrong Format (Commands and Agents)

Commands and agents use JSON array syntax, not space-delimited strings.

**Wrong** - Space-delimited:
```yaml
allowed-tools: Read Write Bash
tools: Read Write Grep
```

**Correct** - JSON arrays:
```yaml
allowed-tools: ["Read", "Write", "Bash"]
tools: ["Read", "Write", "Grep"]
```

Note: Skills use space-delimited (`allowed-tools: Read Write Bash`), but commands and agents use JSON arrays.

## 11. Missing YAML Frontmatter in Commands/Agents

Commands and agents require YAML frontmatter.

**Wrong** - Starting with markdown:
```markdown
# My Command

Instructions here...
```

**Correct** - Frontmatter first:
```yaml
---
description: What this command does
---

# My Command

Instructions here...
```

## 12. Hooks JSON Missing Wrapper Structure

Plugin hooks.json requires a `"hooks"` wrapper key with arrays at each level.

**Wrong** - Direct events (no wrapper):
```json
{
  "PreToolUse": {
    "type": "command",
    "command": "..."
  }
}
```

**Correct** - Full structure with arrays:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "..."
          }
        ]
      }
    ]
  }
}
```

Events map to arrays of matcher groups, each containing an array of hooks.

## 13. README Not at Plugin Root

README.md should be at the plugin root, not inside .claude-plugin.

**Wrong**: `my-plugin/.claude-plugin/README.md`

**Correct**: `my-plugin/README.md`

## 14. Using Command Hooks Instead of Prompt Hooks

Prompt-based hooks are simpler for most validation logic. Reserve command hooks for deterministic checks.

**Wrong** - Complex bash script for a decision Claude can reason about.

**Correct** - Prompt-based hook:
```json
{
  "type": "prompt",
  "prompt": "Verify task completion: tests run, build succeeded. Return 'approve' or 'block'."
}
```

## 15. Not Using ${CLAUDE_PLUGIN_ROOT} in Settings Paths

Settings file paths should be relative to the project, not the plugin.

**Wrong**: `STATE_FILE="${CLAUDE_PLUGIN_ROOT}/.claude/my-plugin.local.md"`

**Correct**: `STATE_FILE=".claude/my-plugin.local.md"`

Use `${CLAUDE_PLUGIN_ROOT}` only for plugin-internal files (hook scripts, etc.).

## 16. Forgetting Restart Requirement for Hooks

Hooks load at session start and cannot be hot-swapped. After editing hooks.json, exit and restart Claude Code for changes to take effect.

## 17. Settings Files Not in .gitignore

Settings files are user-local and should not be committed.

```gitignore
.claude/*.local.md
.claude/*.local.json
```

## 18. MCP Configuration Duplication

Don't configure MCP servers in both `.mcp.json` AND `plugin.json`. Choose one location:

- `.mcp.json`: Recommended for multiple servers
- `plugin.json` `mcpServers` field: Good for a single server

## 19. Missing MCP Tool Naming Convention

MCP tools use the full prefixed name: `mcp__plugin_<plugin-name>_<server-name>__<tool-name>`

**Wrong**: `allowed-tools: ["asana_create_task"]`

**Correct**: `allowed-tools: ["mcp__plugin_asana_asana__asana_create_task"]`

Discover tool names with the `/mcp` command.

## 20. MCP Credentials in Configuration

Never hardcode credentials in MCP config. Use environment variables.

**Wrong**: `"Authorization": "Bearer abc123secret456"`

**Correct**: `"Authorization": "Bearer ${API_TOKEN}"`

Document required environment variables in README.

## 21. Hook Matchers Without Proper Regex

Hook matchers use regex patterns.

**Wrong**: `"matcher": "Write Edit"` (space has no special meaning in regex)

**Correct**: `"matcher": "Write|Edit"` (pipe means OR)

Common patterns: `"Write|Edit"`, `"mcp__.*__delete.*"`, `"*"` (match all).

## 22. Command Instructions TO User Instead of FOR Claude

Commands should tell Claude what to do, not tell the user what to do. See [TEMPLATES.md](TEMPLATES.md) for detailed examples.

**Wrong**: "Please run `npm test` and review the output."

**Correct**: "Execute tests using Bash: `npm test`. Parse output. Report results to user."

## 23. Missing Timeout in Long-Running Hooks

Command hooks need appropriate timeouts.

Recommended timeouts:
- Quick checks: 5-10s
- File operations: 10-30s
- Build/test operations: 60-120s
- Network requests: 30-60s

## 24. Agent Triggering Examples Missing Context

Agent `<example>` tags need a Context field.

**Wrong**: `<example>user: "review my code"\nassistant: "I'll review it"</example>`

**Correct**: `<example>Context: User wants code review\nuser: "review my code"\nassistant: "I'll use code-reviewer for analysis"</example>`

Include both proactive and reactive examples.

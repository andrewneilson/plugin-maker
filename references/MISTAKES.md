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

**Wrong**:
```
my-plugin/                    # Directory name
└── .claude-plugin/
    └── plugin.json           # Contains "name": "myPlugin"
```

**Correct**:
```
my-plugin/                    # Directory name
└── .claude-plugin/
    └── plugin.json           # Contains "name": "my-plugin"
```

## 4. Minimal vs Full plugin.json

The `author` field is optional but helpful for attribution.

**Minimal** - Only required fields:
```json
{
  "name": "my-plugin",
  "description": "My plugin"
}
```

**Full** - With optional author:
```json
{
  "name": "my-plugin",
  "description": "My plugin",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  }
}
```

## 5. Wrong Directory Names (Singular vs Plural)

Component directories must use the correct plural forms.

**Wrong** - Singular names:
```
my-plugin/
├── command/              # Wrong: should be 'commands'
│   └── my-command.md
├── agent/                # Wrong: should be 'agents'
│   └── my-agent.md
└── skill/                # Wrong: should be 'skills'
    └── my-skill/
        └── SKILL.md
```

**Correct** - Plural names:
```
my-plugin/
├── commands/             # Correct: plural
│   └── my-command.md
├── agents/               # Correct: plural
│   └── my-agent.md
└── skills/               # Correct: plural
    └── my-skill/
        └── SKILL.md
```

**Exception**: The `hooks/` directory is conventionally singular.

## 6. Skill Not in Subdirectory

Skills must be in their own subdirectory within `skills/`.

**Wrong** - SKILL.md directly in skills/:
```
my-plugin/
└── skills/
    └── SKILL.md          # Wrong: needs subdirectory
```

**Correct** - SKILL.md in named subdirectory:
```
my-plugin/
└── skills/
    └── my-skill/         # Subdirectory with skill name
        └── SKILL.md      # SKILL.md inside
```

## 7. Hooks Missing Handler Files

Hooks reference scripts that don't exist.

**Wrong** - Handler doesn't exist:
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
But `hooks/handler.py` doesn't exist!

**Correct** - Create the handler file:
```
my-plugin/
└── hooks/
    ├── hooks.json
    └── handler.py        # Must exist and be executable
```

## 8. Agent Description Without Trigger Examples

Agent descriptions should include Examples tags for complex use cases.

**Wrong** - Generic description:
```yaml
---
name: code-analyzer
description: Analyzes code
---
```

**Correct** - Description with trigger scenarios and examples:
```yaml
---
name: code-analyzer
description: Use this agent for deep code analysis including pattern detection and issue identification. Examples: <example>Context: User wants security review\nuser: "Check auth.js for vulnerabilities"\nassistant: "I'll use code-analyzer for security analysis"</example>
---
```

## 9. Command Without $ARGUMENTS Handling

Commands that accept arguments should document how to use them.

**Wrong** - No mention of arguments:
```yaml
---
description: Greet the user
argument-hint: [name]
---

# Greet

Say hello to the user.
```

**Correct** - Documents argument usage:
```yaml
---
description: Greet the user
argument-hint: [name]
---

# Greet

Say hello to the user.

If the user provides a name in $ARGUMENTS, use that name in the greeting.
Otherwise, ask for their name.
```

## 10. Hardcoded Paths in Hooks

Hook commands must use `${CLAUDE_PLUGIN_ROOT}` variable.

**Wrong** - Hardcoded path:
```json
{
  "hooks": {
    "PreToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "python3 /Users/me/.claude/plugins/my-plugin/hooks/handler.py"
      }]
    }]
  }
}
```

**Correct** - Use variable:
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

## 11. Command allowed-tools Wrong Format

Commands use JSON array syntax for allowed-tools, not space-delimited.

**Wrong** - Space-delimited (skill format):
```yaml
---
description: My command
allowed-tools: Read Write Bash
---
```

**Correct** - JSON array (command format):
```yaml
---
description: My command
allowed-tools: ["Read", "Write", "Bash"]
---
```

Note: Skills use space-delimited (`allowed-tools: Read Write Bash`), but commands use JSON arrays.

## 12. Missing YAML Frontmatter in Commands/Agents

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

## 13. Agent tools Field Wrong Type

Agent tools should be an array, not a space-delimited string.

**Wrong**:
```yaml
---
name: my-agent
description: My agent
tools: Read Write Grep
---
```

**Correct**:
```yaml
---
name: my-agent
description: My agent
tools: ["Read", "Write", "Grep"]
---
```

## 14. Hooks JSON Missing Wrapper Structure

The hooks.json structure requires specific nesting.

**Wrong** - Missing wrapper:
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

## 15. README Not at Plugin Root

README.md should be at the plugin root, not inside .claude-plugin.

**Wrong**:
```
my-plugin/
└── .claude-plugin/
    ├── plugin.json
    └── README.md         # Wrong location
```

**Correct**:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── README.md             # At plugin root
```

## 16. Using Command Hooks Instead of Prompt Hooks

Prompt-based hooks are simpler and more flexible for most use cases.

**Wrong** - Unnecessary bash script:
```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/check-completion.sh"
      }]
    }]
  }
}
```

Then create complex bash script to check completion.

**Correct** - Use prompt-based hook:
```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "prompt",
        "prompt": "Verify task completion: tests run, build succeeded, questions answered. Return 'approve' or 'block'."
      }]
    }]
  }
}
```

Reserve command hooks for deterministic checks or when bash is truly needed.

## 17. Missing Hooks Wrapper in plugin.json

Plugin hooks.json requires a wrapper structure.

**Wrong** - Direct events (settings format):
```json
{
  "PreToolUse": [{
    "matcher": "*",
    "hooks": [{"type": "prompt", "prompt": "..."}]
  }]
}
```

**Correct** - Wrapped in "hooks" key (plugin format):
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{"type": "prompt", "prompt": "..."}]
    }]
  }
}
```

## 18. Not Using ${CLAUDE_PLUGIN_ROOT} in Settings Paths

Settings file paths should be relative to project, not plugin.

**Wrong** - Using CLAUDE_PLUGIN_ROOT for settings:
```bash
STATE_FILE="${CLAUDE_PLUGIN_ROOT}/.claude/my-plugin.local.md"
```

**Correct** - Relative to project directory:
```bash
STATE_FILE=".claude/my-plugin.local.md"
```

Use `${CLAUDE_PLUGIN_ROOT}` only for plugin-internal files (hook scripts, etc.).

## 19. Forgetting Restart Requirement for Hooks

Hooks load at session start and cannot be hot-swapped.

**Wrong documentation**:
```markdown
Edit hooks/hooks.json to change behavior.
```

**Correct documentation**:
```markdown
Edit hooks/hooks.json to change behavior.
After editing, restart Claude Code:
1. Exit Claude Code
2. Run `claude` again
3. New hooks will be loaded
```

## 20. Settings Files Not in .gitignore

Settings files are user-local and should not be committed.

**Wrong** - No gitignore:
```
project/
└── .claude/
    └── my-plugin.local.md  # Gets committed!
```

**Correct** - Add to .gitignore:
```gitignore
.claude/*.local.md
.claude/*.local.json
```

Document this in plugin README.

## 21. MCP Configuration Duplication

Don't configure MCP servers in both .mcp.json AND plugin.json.

**Wrong** - Duplicate configuration:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # Has mcpServers field
└── .mcp.json            # Also configures servers
```

**Correct** - Choose one location:

Option 1: Use .mcp.json (recommended for multiple servers):
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # No mcpServers field
└── .mcp.json            # All MCP config here
```

Option 2: Use inline (good for single server):
```json
{
  "name": "my-plugin",
  "mcpServers": {
    "my-server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/server.js"
    }
  }
}
```

## 22. Hardcoded MCP Server Paths

MCP server paths must use ${CLAUDE_PLUGIN_ROOT}.

**Wrong** - Hardcoded path:
```json
{
  "my-server": {
    "command": "/Users/me/.claude/plugins/my-plugin/servers/server.js"
  }
}
```

**Correct** - Use variable:
```json
{
  "my-server": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/server.js"
  }
}
```

## 23. Missing MCP Tool Naming Convention

When referencing MCP tools, use the full prefixed name.

**Wrong** - Tool short name:
```yaml
---
description: Create task
allowed-tools: ["asana_create_task"]
---
```

**Correct** - Full MCP tool name:
```yaml
---
description: Create task
allowed-tools: ["mcp__plugin_asana_asana__asana_create_task"]
---
```

Discover tool names with `/mcp` command.

## 24. MCP Server Command Without Node Buffer

Python MCP servers need PYTHONUNBUFFERED.

**Wrong** - No unbuffered flag:
```json
{
  "my-python-server": {
    "command": "python",
    "args": ["-m", "my_mcp_server"]
  }
}
```

**Correct** - Set PYTHONUNBUFFERED:
```json
{
  "my-python-server": {
    "command": "python",
    "args": ["-m", "my_mcp_server"],
    "env": {
      "PYTHONUNBUFFERED": "1"
    }
  }
}
```

## 25. Using HTTP Instead of HTTPS for MCP

Network-based MCP servers must use secure connections.

**Wrong** - Insecure HTTP:
```json
{
  "api": {
    "type": "http",
    "url": "http://api.example.com/mcp"
  }
}
```

**Correct** - Secure HTTPS:
```json
{
  "api": {
    "type": "https",
    "url": "https://api.example.com/mcp"
  }
}
```

Same for WebSocket: use `wss://` not `ws://`.

## 26. MCP Credentials in Configuration

Never hardcode credentials in MCP config.

**Wrong** - Hardcoded token:
```json
{
  "api": {
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "Authorization": "Bearer abc123secret456"
    }
  }
}
```

**Correct** - Use environment variable:
```json
{
  "api": {
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

Document required environment variables in README.

## 27. Hook Matchers Without Proper Regex

Hook matchers should use valid regex patterns.

**Wrong** - Invalid regex:
```json
{
  "PreToolUse": [{
    "matcher": "Write Edit",  // Space means nothing in regex
    "hooks": [{"type": "prompt", "prompt": "..."}]
  }]
}
```

**Correct** - Proper regex:
```json
{
  "PreToolUse": [{
    "matcher": "Write|Edit",  // Pipe means OR in regex
    "hooks": [{"type": "prompt", "prompt": "..."}]
  }]
}
```

Common patterns:
- `"Write|Edit"` - Match Write OR Edit
- `"mcp__.*__delete.*"` - Match any MCP delete tools
- `"*"` - Match all tools

## 28. Command Instructions TO User Instead of FOR Claude

Commands should tell Claude what to do, not tell the user what to do.

**Wrong** - Instructions TO user:
```yaml
---
description: Run tests
---

# Test Command

Please run the following command:
```bash
npm test
```

Review the output and fix any failures.
```

**Correct** - Instructions FOR Claude:
```yaml
---
description: Run tests and report results
---

# Test Command

When the user runs this command:

1. Execute tests using Bash: `npm test`
2. Parse test output
3. Report results to user:
   - Number passed/failed
   - Details of failures
   - Suggestions for fixes
```

## 29. Missing Timeout in Long-Running Hooks

Command hooks need appropriate timeouts for their workload.

**Wrong** - Default timeout for slow operation:
```json
{
  "SessionStart": [{
    "hooks": [{
      "type": "command",
      "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/slow-setup.sh"
      // No timeout specified, uses default 60s
    }]
  }]
}
```

**Correct** - Explicit timeout:
```json
{
  "SessionStart": [{
    "hooks": [{
      "type": "command",
      "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/slow-setup.sh",
      "timeout": 120  // Longer timeout for slow operation
    }]
  }]
}
```

Recommended timeouts:
- Quick checks: 5-10s
- File operations: 10-30s
- Build/test operations: 60-120s
- Network requests: 30-60s

## 30. Agent Triggering Examples Missing Context

Agent triggering examples need Context field.

**Wrong** - No context:
```yaml
---
description: Examples: <example>user: "review my code"\nassistant: "I'll review it"</example>
---
```

**Correct** - With context:
```yaml
---
description: Examples: <example>Context: User wants code review\nuser: "review my code"\nassistant: "I'll use code-reviewer agent for comprehensive analysis"</example>
---
```

Include both proactive and reactive examples:
- Proactive: Claude suggests the agent
- Reactive: User directly requests agent's capability

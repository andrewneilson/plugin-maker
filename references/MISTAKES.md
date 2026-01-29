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

## 16. Using Command Hooks Instead of Prompt-Based

Prompt-based hooks are often better for context-aware decisions.

**Wrong** - Complex bash script for validation:
```json
{
  "hooks": {
    "PreToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/complex-validator.sh"
      }]
    }]
  }
}
```

**Correct** - Prompt-based for context awareness:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "prompt",
        "prompt": "Validate file write safety. Check: system paths, credentials, path traversal. Return 'approve' or 'deny'.",
        "timeout": 30
      }]
    }]
  }
}
```

## 17. Missing Hook Wrapper in Plugin

Plugin hooks need wrapper format with `"hooks"` field.

**Wrong** - Direct format (this is for settings):
```json
{
  "PreToolUse": [...]
}
```

**Correct** - Wrapper format for plugins:
```json
{
  "hooks": {
    "PreToolUse": [...]
  }
}
```

## 18. Agent Description Without Examples

Agents trigger better with concrete examples.

**Wrong** - No examples:
```yaml
---
name: code-analyzer
description: Use for code analysis
---
```

**Correct** - With examples:
```yaml
---
name: code-analyzer
description: Use this agent for code analysis and issue detection. Examples: <example>Context: User wants security review\nuser: "Check auth.js for vulnerabilities"\nassistant: "I'll use code-analyzer for security analysis"</example>
---
```

## 19. Wrong MCP Server Type for Use Case

Choosing the wrong server type for the use case.

**Wrong** - stdio for cloud service:
```json
{
  "mcpServers": {
    "github": {
      "command": "some-local-command"
    }
  }
}
```

**Correct** - SSE for cloud service with OAuth:
```json
{
  "mcpServers": {
    "github": {
      "type": "sse",
      "url": "https://mcp.github.com/sse"
    }
  }
}
```

## 20. Agent Description in Second Person

Use third-person for agent descriptions.

**Wrong** - Second person:
```yaml
description: Use me when you want to analyze code
```

**Correct** - Third person:
```yaml
description: Use this agent when analyzing code for security issues
```

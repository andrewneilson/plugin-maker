# Hook Patterns and Examples

Comprehensive patterns for implementing hooks in Claude Code plugins.

## Hook Types

### Prompt-Based Hooks (Recommended for Context-Aware Logic)

Use LLM-driven decision making for flexible, context-aware validation.

**Benefits**:
- Natural language reasoning
- Better edge case handling
- No bash scripting required
- Easier to maintain and extend

**Example - PreToolUse validation**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate file write safety. Check: system paths, credentials, path traversal, sensitive content. Return 'approve' or 'deny'.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**Example - Stop hook for completeness**:
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify task completion: tests run, build succeeded, questions answered, documentation updated. Return 'approve' to stop or 'block' with reason to continue."
          }
        ]
      }
    ]
  }
}
```

### Command Hooks (For Deterministic Checks)

Execute bash commands for fast, deterministic validations.

**Use for**:
- File system operations
- External tool integrations
- Performance-critical checks
- Deterministic validations

**Example - Python hook handler**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/validate-bash.py",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

## Hook Events Reference

### PreToolUse

Execute before any tool runs. Approve, deny, or modify tool calls.

**Input variables**:
- `$TOOL_NAME` - Tool being called
- `$TOOL_INPUT` - Tool input parameters

**Output format**:
```json
{
  "hookSpecificOutput": {
    "permissionDecision": "allow|deny|ask",
    "updatedInput": {"field": "modified_value"}
  },
  "systemMessage": "Explanation for Claude"
}
```

**Example - Block dangerous operations**:
```python
#!/usr/bin/env python3
import json
import sys

def main():
    input_data = json.load(sys.stdin)
    tool_name = input_data.get("tool_name", "")
    tool_input = input_data.get("tool_input", {})

    # Block rm -rf on root paths
    if tool_name == "Bash":
        command = tool_input.get("command", "")
        if "rm -rf" in command and ("/" == command.strip()[-1]):
            print(json.dumps({
                "decision": "block",
                "reason": "Blocking potentially dangerous rm -rf on root path"
            }))
            return

    # Allow by default
    print(json.dumps({}))

if __name__ == "__main__":
    main()
```

### PostToolUse

Execute after tool completes. React to results, provide feedback, or log.

**Input variables**:
- `$TOOL_NAME` - Tool that was called
- `$TOOL_INPUT` - Tool input parameters
- `$TOOL_RESULT` - Tool output/result

**Example - Analyze edit results**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Analyze edit result for potential issues: syntax errors, security vulnerabilities, breaking changes. Provide feedback if issues detected."
          }
        ]
      }
    ]
  }
}
```

### Stop

Execute when main agent considers stopping. Validate completeness.

**Input variables**:
- `$REASON` - Why Claude wants to stop

**Output format**:
```json
{
  "decision": "approve|block",
  "reason": "Explanation",
  "systemMessage": "Additional context for Claude"
}
```

**Example - Ensure tests ran**:
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Before stopping, verify: 1) All tests passed, 2) Build succeeded, 3) Documentation updated, 4) User questions answered. Return 'approve' if complete, 'block' with specific missing items if not."
          }
        ]
      }
    ]
  }
}
```

### SubagentStop

Execute when subagent considers stopping. Ensure subagent completed its task.

Same format as Stop hook, but for subagent completion validation.

### UserPromptSubmit

Execute when user submits a prompt. Add context, validate, or block prompts.

**Input variables**:
- `$USER_PROMPT` - The user's submitted prompt

**Example - Add security context**:
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if prompt discusses auth, permissions, or API security. If yes, return relevant security best practices and warnings as systemMessage."
          }
        ]
      }
    ]
  }
}
```

### SessionStart

Execute when Claude Code session begins. Load context and set environment.

**Special capability**: Persist environment variables using `$CLAUDE_ENV_FILE`:

```bash
#!/bin/bash
# Load project context
echo "export PROJECT_TYPE=nodejs" >> "$CLAUDE_ENV_FILE"
echo "export API_ENDPOINT=https://api.example.com" >> "$CLAUDE_ENV_FILE"

# Detect package manager
if [ -f "package-lock.json" ]; then
  echo "export PKG_MANAGER=npm" >> "$CLAUDE_ENV_FILE"
elif [ -f "yarn.lock" ]; then
  echo "export PKG_MANAGER=yarn" >> "$CLAUDE_ENV_FILE"
fi
```

**Example configuration**:
```json
{
  "hooks": {
    "SessionStart": [
      {
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

### SessionEnd

Execute when session ends. Cleanup, logging, and state preservation.

**Example - Save session summary**:
```bash
#!/bin/bash
# Save session summary
echo "Session ended at $(date)" >> ~/.claude/session-log.txt
```

### PreCompact

Execute before context compaction. Add critical information to preserve.

**Example - Preserve key context**:
```json
{
  "hooks": {
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Summarize key decisions, open questions, and critical context that must be preserved during compaction."
          }
        ]
      }
    ]
  }
}
```

### Notification

Execute when Claude sends notifications. React to user notifications.

## Matchers

Control which tools trigger your hooks.

### Exact Match
```json
"matcher": "Write"
```

### Multiple Tools (OR)
```json
"matcher": "Read|Write|Edit"
```

### Wildcard (All Tools)
```json
"matcher": "*"
```

### Regular Expression
```json
"matcher": "Write.*|Edit.*"
```

## Hook Configuration Formats

### Plugin Format (hooks/hooks.json)

**Requires wrapper with "hooks" field**:

```json
{
  "description": "Brief explanation of hooks (optional)",
  "hooks": {
    "PreToolUse": [...],
    "Stop": [...],
    "SessionStart": [...]
  }
}
```

### Settings Format (.claude/settings.json)

**Direct format without wrapper**:

```json
{
  "PreToolUse": [...],
  "Stop": [...],
  "SessionStart": [...]
}
```

## Environment Variables

Available in all command hooks:

- `$CLAUDE_PROJECT_DIR` - Project root path
- `$CLAUDE_PLUGIN_ROOT` - Plugin directory (use for portable paths)
- `$CLAUDE_ENV_FILE` - SessionStart only: persist env vars here
- `$CLAUDE_CODE_REMOTE` - Set if running in remote context

**Always use ${CLAUDE_PLUGIN_ROOT} for portability**:

```json
{
  "type": "command",
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

## Complete Examples

### Security Validation Plugin

```json
{
  "description": "Security validation hooks for safe operations",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit|Create",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate file write safety. Check for: 1) System paths (/etc, /usr), 2) Credentials in content, 3) Path traversal attempts, 4) Sensitive file patterns (.env, id_rsa). Return 'approve' for safe operations or 'deny' with reason."
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/bash-validator.py",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Analyze the edit for security issues: SQL injection patterns, XSS vulnerabilities, hardcoded credentials, insecure crypto. Provide warnings if issues detected."
          }
        ]
      }
    ]
  }
}
```

### Quality Assurance Plugin

```json
{
  "description": "Enforce quality standards before completion",
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify task completion checklist: 1) All tests passing, 2) Build successful, 3) Linting clean, 4) Documentation updated, 5) User questions answered. Return 'approve' if all complete, 'block' with missing items if not."
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify subagent completed its assigned task. Return 'approve' if task done, 'block' with reason if incomplete."
          }
        ]
      }
    ]
  }
}
```

### Project Context Plugin

```json
{
  "description": "Load project context at session start",
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/load-context.sh",
            "timeout": 15
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "If user asks about project setup, build process, or architecture, provide context from project README and docs."
          }
        ]
      }
    ]
  }
}
```

## Best Practices

1. **Use prompt-based hooks for context-aware decisions**: They handle edge cases better than scripts
2. **Use command hooks for deterministic checks**: Fast file system operations, validation
3. **Always use ${CLAUDE_PLUGIN_ROOT}**: Ensures portability across installations
4. **Set appropriate timeouts**: 10-30 seconds for prompts, 5-10 for commands
5. **Use matchers to reduce noise**: Target specific tools, not everything
6. **Provide helpful system messages**: Explain decisions to Claude
7. **Test hooks thoroughly**: Use `--plugin-dir` for rapid iteration
8. **Handle errors gracefully**: Return empty JSON to allow by default

## Debugging Hooks

```bash
# Test plugin with debug output
claude --debug --plugin-dir ./my-plugin

# Check hook execution
tail -f ~/.claude/logs/hooks.log

# Validate hooks.json
cat ./my-plugin/hooks/hooks.json | jq .
```

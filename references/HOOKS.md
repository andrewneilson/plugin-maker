# Hook Development Guide

Comprehensive guide for creating event-driven automation hooks in Claude Code plugins.

## Overview

Hooks are event-driven scripts that execute in response to Claude Code lifecycle events. They enable validation, context injection, policy enforcement, and workflow automation.

## Hook Types

### Prompt-Based Hooks (Recommended)

Use LLM reasoning for context-aware decisions:

```json
{
  "type": "prompt",
  "prompt": "Evaluate if this tool use is appropriate: $TOOL_INPUT",
  "timeout": 30
}
```

**Benefits:**
- Natural language reasoning
- Context-aware decisions
- No bash scripting required
- Better edge case handling

**Supported events:** PreToolUse, UserPromptSubmit, Stop, SubagentStop

### Command Hooks

Execute bash commands for deterministic checks:

```json
{
  "type": "command",
  "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/handler.py",
  "timeout": 10
}
```

**Use for:**
- Fast deterministic validations
- File system operations
- External tool integrations

## Hook Events

### PreToolUse

Execute before any tool runs. Validate, modify, or block tool calls.

**Example:**
```json
{
  "PreToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Validate file write safety. Check: path traversal, credentials, sensitive files. Return 'approve' or 'deny'."
        }
      ]
    }
  ]
}
```

### PostToolUse

Execute after tool completes. React to results, provide feedback, or log.

**Example:**
```json
{
  "PostToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/log-command.sh"
        }
      ]
    }
  ]
}
```

### Stop

Execute when main agent considers stopping. Validate completeness.

**Example:**
```json
{
  "Stop": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Verify task completion: tests run, build succeeded, questions answered. Return 'approve' to stop or 'block' with reason."
        }
      ]
    }
  ]
}
```

### SubagentStop

Execute when subagent considers stopping.

Similar to Stop hook, but for subagents.

### UserPromptSubmit

Execute when user submits a prompt. Add context or validate input.

**Example:**
```json
{
  "UserPromptSubmit": [
    {
      "matcher": "*",
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Check if prompt requires security guidance. If discussing auth or permissions, return relevant warnings."
        }
      ]
    }
  ]
}
```

### SessionStart

Execute when Claude Code session begins. Load context and initialize.

**Example:**
```json
{
  "SessionStart": [
    {
      "matcher": "*",
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
```

**Special capability:** Persist environment variables using `$CLAUDE_ENV_FILE`:
```bash
echo "export PROJECT_TYPE=nodejs" >> "$CLAUDE_ENV_FILE"
```

### SessionEnd

Execute when session ends. Cleanup and state preservation.

### PreCompact

Execute before context compaction. Add critical information to preserve.

### Notification

Execute when Claude sends notifications. React to user notifications.

## Hook Configuration

### Plugin Format

In `hooks/hooks.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate file write safety"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify task completion"
          }
        ]
      }
    ]
  }
}
```

**Important:** Plugin hooks require wrapper with `"hooks"` key.

### Matchers

Control which tools trigger hooks:

```json
// Exact match
"matcher": "Write"

// Multiple tools (regex)
"matcher": "Read|Write|Edit"

// All tools
"matcher": "*"

// MCP tools
"matcher": "mcp__.*__delete.*"
```

## Environment Variables

Available in command hooks:

- `$CLAUDE_PROJECT_DIR` - Project root path
- `$CLAUDE_PLUGIN_ROOT` - Plugin directory (use for portable paths)
- `$CLAUDE_ENV_FILE` - SessionStart only: persist environment variables
- `$CLAUDE_CODE_REMOTE` - Set if running in remote context

**Always use ${CLAUDE_PLUGIN_ROOT} for portable paths:**

```json
{
  "type": "command",
  "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/validate.sh"
}
```

## Hook Input Format

Hooks receive JSON via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.txt",
  "cwd": "/current/working/dir",
  "permission_mode": "ask|allow",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {"file_path": "/path/to/file"}
}
```

Access fields in prompts: `$TOOL_INPUT`, `$TOOL_RESULT`, `$USER_PROMPT`

## Hook Output Format

### Standard Output

```json
{
  "continue": true,
  "suppressOutput": false,
  "systemMessage": "Message for Claude"
}
```

### PreToolUse Decision

```json
{
  "hookSpecificOutput": {
    "permissionDecision": "allow|deny|ask",
    "updatedInput": {"field": "modified_value"}
  },
  "systemMessage": "Explanation for Claude"
}
```

### Stop Decision

```json
{
  "decision": "approve|block",
  "reason": "Explanation",
  "systemMessage": "Additional context"
}
```

### Exit Codes

- `0` - Success (stdout shown in transcript)
- `2` - Blocking error (stderr fed back to Claude)
- Other - Non-blocking error

## Security Best Practices

### Input Validation

Always validate inputs:

```bash
#!/bin/bash
set -euo pipefail

input=$(cat)
tool_name=$(echo "$input" | jq -r '.tool_name')

# Validate tool name format
if [[ ! "$tool_name" =~ ^[a-zA-Z0-9_]+$ ]]; then
  echo '{"decision": "deny", "reason": "Invalid tool name"}' >&2
  exit 2
fi
```

### Path Safety

Check for path traversal:

```bash
file_path=$(echo "$input" | jq -r '.tool_input.file_path')

# Deny path traversal
if [[ "$file_path" == *".."* ]]; then
  echo '{"decision": "deny", "reason": "Path traversal detected"}' >&2
  exit 2
fi

# Deny sensitive files
if [[ "$file_path" == *".env"* ]]; then
  echo '{"decision": "deny", "reason": "Sensitive file"}' >&2
  exit 2
fi
```

### Quote Variables

```bash
# GOOD: Quoted
echo "$file_path"
cd "$CLAUDE_PROJECT_DIR"

# BAD: Unquoted (injection risk)
echo $file_path
cd $CLAUDE_PROJECT_DIR
```

## Performance Considerations

### Parallel Execution

All matching hooks run **in parallel**:

```json
{
  "PreToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        {"type": "command", "command": "check1.sh"},  // Parallel
        {"type": "command", "command": "check2.sh"},  // Parallel
        {"type": "prompt", "prompt": "Validate..."}   // Parallel
      ]
    }
  ]
}
```

**Design implications:**
- Hooks don't see each other's output
- Non-deterministic ordering
- Design for independence

### Optimization Tips

1. Use command hooks for quick deterministic checks
2. Use prompt hooks for complex reasoning
3. Set appropriate timeouts (default: command 60s, prompt 30s)
4. Cache validation results

## Temporarily Active Hooks

Create hooks that activate conditionally:

```bash
#!/bin/bash
# Only active when flag file exists
FLAG_FILE="$CLAUDE_PROJECT_DIR/.enable-strict-validation"

if [ ! -f "$FLAG_FILE" ]; then
  exit 0  # Not enabled, skip
fi

# Flag present, run validation
input=$(cat)
# ... validation logic ...
```

See [PLUGIN-SETTINGS.md](PLUGIN-SETTINGS.md) for configuration-driven activation.

## Hook Lifecycle

### Load at Session Start

**Important:** Hooks load when Claude Code starts. Changes require restart.

**Cannot hot-swap:**
- Editing `hooks/hooks.json` won't affect current session
- Must restart Claude Code: exit and run `claude` again

**To test changes:**
1. Edit hook configuration
2. Exit Claude Code
3. Restart: `claude` or `cc`
4. Test with `claude --debug`

## Debugging Hooks

### Enable Debug Mode

```bash
claude --debug
```

Look for hook registration, execution logs, and timing.

### Test Hook Scripts

Test command hooks directly:

```bash
echo '{"tool_name": "Write", "tool_input": {"file_path": "/test"}}' | \
  bash hooks/handler.sh

echo "Exit code: $?"
```

### Validate JSON Output

```bash
output=$(./hook.sh < test-input.json)
echo "$output" | jq .
```

## Quick Reference

### Event Summary

| Event | When | Use For |
|-------|------|---------|
| PreToolUse | Before tool | Validation, modification |
| PostToolUse | After tool | Feedback, logging |
| UserPromptSubmit | User input | Context, validation |
| Stop | Agent stopping | Completeness check |
| SubagentStop | Subagent done | Task validation |
| SessionStart | Session begins | Context loading |
| SessionEnd | Session ends | Cleanup, logging |
| PreCompact | Before compact | Preserve context |
| Notification | User notified | Logging, reactions |

### Best Practices

**DO:**
- ✅ Use prompt-based hooks for complex logic
- ✅ Use ${CLAUDE_PLUGIN_ROOT} for portability
- ✅ Validate all inputs
- ✅ Quote bash variables
- ✅ Set appropriate timeouts
- ✅ Return structured JSON

**DON'T:**
- ❌ Use hardcoded paths
- ❌ Trust input without validation
- ❌ Create long-running hooks
- ❌ Rely on hook execution order
- ❌ Log sensitive information

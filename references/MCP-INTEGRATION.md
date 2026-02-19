# MCP Integration Guide

Comprehensive guide for integrating Model Context Protocol (MCP) servers into Claude Code plugins.

## Overview

MCP integration enables plugins to connect external services and APIs as native tools in Claude Code. Use MCP to expose 10+ tools from a single service, handle complex authentication, and bundle external integrations with your plugin.

## Configuration Methods

### Method 1: Dedicated .mcp.json (Recommended)

Create `.mcp.json` at plugin root:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

**Benefits:**
- Clear separation of concerns
- Easier to maintain
- Better for multiple servers

### Method 2: Inline in plugin.json

Add `mcpServers` field to plugin.json:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

**Benefits:**
- Single configuration file
- Good for simple single-server plugins

## MCP Server Types

### stdio (Local Process)

Execute local MCP servers as child processes. Best for local tools and custom servers.

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"],
    "env": {
      "LOG_LEVEL": "debug"
    }
  }
}
```

**Use cases:**
- File system access
- Local database connections
- Custom MCP servers
- NPM-packaged MCP servers

**Process lifecycle:**
- Claude Code spawns and manages the process
- Communicates via stdin/stdout
- Terminates when Claude Code exits

**Best practices:**
- Use absolute paths or ${CLAUDE_PLUGIN_ROOT}
- Set PYTHONUNBUFFERED for Python servers
- Pass configuration via args or env, not stdin
- Log to stderr, not stdout (stdout is for MCP protocol)

### SSE (Server-Sent Events)

Connect to hosted MCP servers with OAuth support. Best for cloud services.

```json
{
  "hosted-service": {
    "type": "sse",
    "url": "https://mcp.example.com/sse",
    "headers": {
      "X-API-Version": "v1"
    }
  }
}
```

**OAuth authentication:**
Claude Code handles OAuth flow automatically:
1. User prompted to authenticate on first use
2. Opens browser for OAuth flow
3. Tokens stored securely
4. Automatic token refresh

**Use cases:**
- Official services (Asana, GitHub)
- Custom hosted servers
- Services requiring OAuth

**Best practices:**
- Always use HTTPS, never HTTP
- Let OAuth handle authentication when available
- Use environment variables for tokens
- Document OAuth scopes required

### HTTP (REST API)

Connect to RESTful MCP servers via standard HTTP requests. Best for token-based auth.

```json
{
  "api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}",
      "Content-Type": "application/json"
    }
  }
}
```

**Use cases:**
- REST API backends
- Internal services
- Microservices
- Serverless functions

**Best practices:**
- Use HTTPS for all connections
- Store tokens in environment variables
- Implement retry logic for transient failures
- Handle rate limiting

### WebSocket (Real-time)

Connect to MCP servers via WebSocket for real-time bidirectional communication.

```json
{
  "realtime": {
    "type": "ws",
    "url": "wss://mcp.example.com/ws",
    "headers": {
      "Authorization": "Bearer ${TOKEN}"
    }
  }
}
```

**Use cases:**
- Real-time data streaming
- Live updates and notifications
- Collaborative editing
- Low-latency tool calls
- Push notifications from server

**Best practices:**
- Always use wss:// (secure WebSocket)
- Implement reconnection logic
- Handle connection state changes
- Use heartbeat messages for keep-alive

## Tool Naming Convention

MCP tools follow this naming pattern:

```
mcp__plugin_<plugin-name>_<server-name>__<tool-name>
```

**Examples:**

Asana plugin with asana server:
- `mcp__plugin_asana_asana__asana_create_task`
- `mcp__plugin_asana_asana__asana_search_tasks`
- `mcp__plugin_asana_asana__asana_get_project`

Custom plugin with database server:
- `mcp__plugin_myplug_database__query`
- `mcp__plugin_myplug_database__execute`
- `mcp__plugin_myplug_database__list_tables`

### Discovering Tool Names

Use the `/mcp` command to see:
- All available MCP servers
- Tools provided by each server
- Tool schemas and descriptions
- Full tool names for use in configuration

## Using MCP Tools in Commands

### Pre-Allowing Tools

Specify MCP tools in command frontmatter:

```markdown
---
description: Create a new Asana task
allowed-tools: [
  "mcp__plugin_asana_asana__asana_create_task"
]
---

# Create Task Command

To create a task:
1. Gather task details from user
2. Use mcp__plugin_asana_asana__asana_create_task with the details
3. Confirm creation to user
```

### Multiple Tools

```markdown
---
allowed-tools: [
  "mcp__plugin_asana_asana__asana_create_task",
  "mcp__plugin_asana_asana__asana_search_tasks",
  "mcp__plugin_asana_asana__asana_get_project"
]
---
```

### Wildcard (Use Sparingly)

```markdown
---
allowed-tools: ["mcp__plugin_asana_asana__*"]
---
```

**Caution:** Only use wildcards if the command truly needs access to all tools from a server.

## Using MCP Tools in Agents

Agents can use MCP tools autonomously without pre-allowing them:

```markdown
---
name: asana-status-updater
description: This agent should be used when the user asks to "update Asana status" or "generate project report"
model: inherit
color: blue
---

## Role

Autonomous agent for generating Asana project status reports.

## Process

1. **Query tasks**: Use mcp__plugin_asana_asana__asana_search_tasks to get all tasks
2. **Analyze progress**: Calculate completion rates and identify blockers
3. **Generate report**: Create formatted status update

## Available Tools

The agent has access to all Asana MCP tools without pre-approval.
```

## Tool Call Patterns

### Pattern 1: Simple Tool Call

```markdown
Steps:
1. Validate user provided required fields
2. Call mcp__plugin_api_server__create_item with validated data
3. Check for errors
4. Display confirmation
```

### Pattern 2: Sequential Tools

```markdown
Steps:
1. Search for existing items: mcp__plugin_api_server__search
2. If not found, create new: mcp__plugin_api_server__create
3. Add metadata: mcp__plugin_api_server__update_metadata
4. Return final item ID
```

### Pattern 3: Error Handling

```markdown
Steps:
1. Try to call mcp__plugin_api_server__get_data
2. If error (rate limit, network, etc.):
   - Wait and retry (max 3 attempts)
   - If still failing, inform user
   - Suggest checking configuration
3. On success, process data
```

## Environment Variables

Available in MCP server configurations:

- `${CLAUDE_PLUGIN_ROOT}` - Plugin directory (use for portable paths)
- `${API_KEY}` - User-configured API keys
- `${DB_URL}` - Database connection strings
- Custom variables defined in `.env` files

**Always use ${CLAUDE_PLUGIN_ROOT} for portable paths:**

```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/servers/my-server",
  "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
}
```

## Lifecycle Management

### Server Startup

1. Claude Code reads `.mcp.json` or `plugin.json`
2. Spawns server processes (stdio) or connects (SSE/HTTP/WebSocket)
3. Performs MCP handshake
4. Discovers available tools
5. Registers tools with Claude Code

### Server Shutdown

- stdio servers: Process terminated when Claude Code exits
- Network servers: Connections closed gracefully
- Resources cleaned up automatically

### Reconnection

- SSE and WebSocket servers support automatic reconnection
- HTTP servers retry on connection failures
- stdio servers restart on crash (with backoff)

## Security Best Practices

### Input Validation

Always validate tool parameters:

```markdown
Steps:
1. Check required parameters:
   - Title is not empty
   - Workspace ID is provided
   - Due date is valid format (YYYY-MM-DD)
2. If validation fails, ask user to provide missing data
3. If validation passes, call MCP tool
```

### Credential Management

**DO:**
- Use environment variables for credentials
- Document required API keys
- Use OAuth when available
- Store tokens securely

**DON'T:**
- Hardcode credentials in configuration
- Log sensitive information
- Share credentials across plugins
- Commit credentials to version control

### Path Safety

Use ${CLAUDE_PLUGIN_ROOT} for portable paths:

```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/servers/server.js",
  "args": ["--data", "${CLAUDE_PLUGIN_ROOT}/data"]
}
```

## Performance Optimization

### Batching Requests

**Good: Single query with filters**
```markdown
Steps:
1. Call mcp__plugin_api_server__search with filters:
   - project_id: "123"
   - status: "active"
   - limit: 100
2. Process all results
```

**Avoid: Many individual queries**
```markdown
Steps:
1. For each item ID:
   - Call mcp__plugin_api_server__get_item
```

### Caching Results

```markdown
Steps:
1. Call expensive MCP operation: mcp__plugin_api_server__analyze
2. Store results in variable for reuse
3. Use cached results for subsequent operations
4. Only re-fetch if data changes
```

### Parallel Tool Calls

When tools don't depend on each other, call in parallel:

```markdown
Steps:
1. Make parallel calls (Claude handles this automatically):
   - mcp__plugin_api_server__get_project
   - mcp__plugin_api_server__get_users
   - mcp__plugin_api_server__get_tags
2. Wait for all to complete
3. Combine results
```

## Error Handling

### User-Friendly Error Messages

**Good error messages:**
```
❌ "Could not create task. Please check:
   1. You're logged into Asana
   2. You have access to workspace 'Engineering'
   3. The project 'Q1 Goals' exists"
```

**Poor error messages:**
```
❌ "Error: MCP tool returned 403"
```

### Common Error Types

| Error | Meaning | Solution |
|-------|---------|----------|
| 401 | Authentication failed | Check credentials, re-authenticate |
| 403 | Permission denied | Verify user has required permissions |
| 429 | Rate limit exceeded | Implement backoff, reduce request rate |
| 500 | Server error | Check server logs, retry later |
| Timeout | Request too slow | Increase timeout, optimize query |

## Testing MCP Integration

### Local Testing

1. **Configure MCP server** in `.mcp.json`
2. **Install plugin locally** in `.claude-plugin/`
3. **Verify tools available** with `/mcp`
4. **Test command** that uses tools
5. **Check debug output**: `claude --debug`

### Test Scenarios

**Test successful calls:**
```markdown
Steps:
1. Create test data in external service
2. Run command that queries this data
3. Verify correct results returned
```

**Test error cases:**
```markdown
Steps:
1. Test with missing authentication
2. Test with invalid parameters
3. Test with non-existent resources
4. Verify graceful error handling
```

## Troubleshooting

### Server Won't Start

**Check:**
- Command exists and is executable
- File paths are correct
- Permissions are set
- Review `claude --debug` logs

### Tools Not Available

**Check:**
- MCP server configured correctly
- Server connected (check `/mcp`)
- Tool names match exactly (case-sensitive)
- Restart Claude Code after config changes

### Tool Calls Failing

**Check:**
- Authentication is valid
- Parameters match tool schema
- Required parameters provided
- Check `claude --debug` logs

### Performance Issues

**Check:**
- Batching queries instead of individual calls
- Caching results when appropriate
- Not making unnecessary tool calls
- Parallel calls when possible

## Best Practices Summary

**DO:**
- ✅ Use ${CLAUDE_PLUGIN_ROOT} for portable paths
- ✅ Store credentials in environment variables
- ✅ Validate all tool parameters
- ✅ Handle errors gracefully
- ✅ Batch requests when possible
- ✅ Document required tools in commands
- ✅ Test with `claude --debug`
- ✅ Use HTTPS/WSS for network connections

**DON'T:**
- ❌ Hardcode credentials
- ❌ Use absolute paths
- ❌ Make unnecessary tool calls
- ❌ Expose internal errors to users
- ❌ Use HTTP (unencrypted) connections
- ❌ Skip error handling
- ❌ Forget to document authentication requirements

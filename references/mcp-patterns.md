# MCP Integration Patterns

Comprehensive patterns for integrating Model Context Protocol (MCP) servers into Claude Code plugins.

## Overview

MCP enables plugins to integrate with external services and APIs by providing structured tool access. Use MCP when you need to:
- Connect to external databases, APIs, or file systems
- Provide 10+ related tools from a single service
- Handle OAuth or complex authentication
- Bundle external service capabilities with your plugin

## Configuration Methods

### Method 1: Dedicated .mcp.json (Recommended)

Create `.mcp.json` at plugin root for clearer separation:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  },
  "api-connector": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/servers/api.js"]
  }
}
```

**Benefits**:
- Clear separation of concerns
- Easier to maintain
- Better for multiple servers

### Method 2: Inline in plugin.json

Add `mcpServers` field to `.claude-plugin/plugin.json`:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Plugin with MCP integration",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

**Benefits**:
- Single configuration file
- Good for simple single-server plugins

## MCP Server Types

### stdio (Local Process)

Execute local MCP servers as child processes. Best for local tools and custom servers.

**Use cases**:
- File system access
- Local database connections
- Custom MCP servers
- NPM-packaged MCP servers

**Process management**:
- Claude Code spawns and manages the process
- Communicates via stdin/stdout
- Terminates when Claude Code exits

**Example - NPM server**:
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

**Example - Python server**:
```json
{
  "data-processor": {
    "command": "python3",
    "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/server.py"],
    "env": {
      "PYTHONPATH": "${CLAUDE_PLUGIN_ROOT}/mcp-server"
    }
  }
}
```

**Example - Custom binary**:
```json
{
  "custom-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/bin/mcp-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.yaml"]
  }
}
```

### SSE (Server-Sent Events)

Connect to hosted MCP servers with OAuth support. Best for cloud services.

**Use cases**:
- Official hosted MCP servers (Asana, GitHub, etc.)
- Cloud services with MCP endpoints
- OAuth-based authentication
- No local installation needed

**Authentication**:
- OAuth flows handled automatically
- User prompted on first use
- Tokens managed by Claude Code

**Example - Asana**:
```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  }
}
```

**Example - GitHub**:
```json
{
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  }
}
```

### HTTP (REST API)

Connect to RESTful MCP servers with token authentication.

**Use cases**:
- REST API-based MCP servers
- Token-based authentication
- Custom API backends
- Stateless interactions

**Example - Token auth**:
```json
{
  "api-service": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}",
      "X-Custom-Header": "value"
    }
  }
}
```

**Example - Basic auth**:
```json
{
  "secured-api": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Basic ${BASE64_CREDS}"
    }
  }
}
```

### WebSocket (Real-time)

Connect to WebSocket MCP servers for real-time bidirectional communication.

**Use cases**:
- Real-time data streams
- Live updates and notifications
- Persistent connections
- Bidirectional communication

**Example - WebSocket server**:
```json
{
  "realtime-data": {
    "type": "ws",
    "url": "wss://realtime.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${WS_TOKEN}"
    }
  }
}
```

## Environment Variable Expansion

All MCP configurations support environment variable expansion.

**In commands and args**:
```json
{
  "my-server": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
  }
}
```

**In env field**:
```json
{
  "my-server": {
    "command": "node",
    "args": ["server.js"],
    "env": {
      "DB_URL": "${DB_URL}",
      "API_KEY": "${MY_API_KEY}",
      "LOG_LEVEL": "debug"
    }
  }
}
```

**Available variables**:
- `${CLAUDE_PLUGIN_ROOT}` - Plugin directory path (always use this for portability)
- `${HOME}` - User home directory
- Any environment variable set by user or system

## MCP Tool Naming

MCP tools appear in Claude with structured names:

**Format**: `mcp__plugin_<plugin-name>_<server-name>__<tool-name>`

**Example**:
- Plugin: `database-tools`
- Server: `postgres`
- Tool: `query`
- Full name: `mcp__plugin_database-tools_postgres__query`

This namespacing prevents conflicts between plugins and servers.

## Authentication Patterns

### OAuth (SSE Servers)

OAuth flows are handled automatically by Claude Code:

```json
{
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  }
}
```

User will be prompted to authenticate on first use. Claude Code manages tokens.

### Token-Based (HTTP Servers)

Use environment variables for tokens:

```json
{
  "api-service": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

User must set `API_TOKEN` environment variable.

### API Keys (stdio Servers)

Pass API keys via environment:

```json
{
  "cloud-service": {
    "command": "npx",
    "args": ["-y", "@example/mcp-server"],
    "env": {
      "API_KEY": "${CLOUD_API_KEY}"
    }
  }
}
```

### Custom Authentication

Implement in your MCP server:

```json
{
  "custom-auth": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/auth-server",
    "args": ["--key", "${SERVICE_KEY}", "--secret", "${SERVICE_SECRET}"]
  }
}
```

## Complete Examples

### Database Plugin

```json
{
  "postgres": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-postgres"],
    "env": {
      "POSTGRES_URL": "${DATABASE_URL}"
    }
  },
  "mongodb": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-mongodb"],
    "env": {
      "MONGO_URL": "${MONGO_URL}"
    }
  }
}
```

**plugin.json**:
```json
{
  "name": "database-tools",
  "version": "1.0.0",
  "description": "Database connectivity via MCP servers"
}
```

**README snippet**:
```markdown
## Setup

Set environment variables:
```bash
export DATABASE_URL="postgresql://localhost/mydb"
export MONGO_URL="mongodb://localhost/mydb"
```
```

### Cloud Services Plugin

```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  },
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  },
  "slack": {
    "type": "http",
    "url": "https://api.slack.com/mcp",
    "headers": {
      "Authorization": "Bearer ${SLACK_TOKEN}"
    }
  }
}
```

### Custom API Plugin

```json
{
  "company-api": {
    "type": "http",
    "url": "https://api.company.com/mcp/v1",
    "headers": {
      "Authorization": "Bearer ${COMPANY_API_TOKEN}",
      "X-Client-ID": "claude-plugin",
      "X-API-Version": "2024-01"
    }
  }
}
```

### File System Plugin

```json
{
  "safe-filesystem": {
    "command": "npx",
    "args": [
      "-y",
      "@modelcontextprotocol/server-filesystem",
      "${CLAUDE_PROJECT_DIR}",
      "${HOME}/Documents"
    ],
    "env": {
      "LOG_LEVEL": "info"
    }
  }
}
```

### Bundled MCP Server Plugin

**Directory structure**:
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
├── mcp-server/
│   ├── package.json
│   ├── index.js
│   └── tools/
│       ├── search.js
│       ├── analyze.js
│       └── report.js
└── README.md
```

**.mcp.json**:
```json
{
  "my-tools": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
    "env": {
      "NODE_ENV": "production",
      "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config.json"
    }
  }
}
```

**mcp-server/package.json**:
```json
{
  "name": "my-plugin-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "index.js",
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  }
}
```

## Best Practices

### 1. Always Use ${CLAUDE_PLUGIN_ROOT}

**Correct**:
```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/servers/server"
}
```

**Incorrect**:
```json
{
  "command": "/Users/me/.claude/plugins/my-plugin/servers/server"
}
```

### 2. Document Required Environment Variables

In README.md:
```markdown
## Setup

This plugin requires the following environment variables:

- `DB_URL` - PostgreSQL connection string
- `API_KEY` - Service API key

Set them before using the plugin:
```bash
export DB_URL="postgresql://localhost/mydb"
export API_KEY="your-key-here"
```
```

### 3. Provide Setup Scripts

**setup.sh**:
```bash
#!/bin/bash
cd "${CLAUDE_PLUGIN_ROOT}/mcp-server"
npm install
```

Reference in README:
```markdown
## Installation

```bash
cd ~/.claude/plugins/my-plugin
bash setup.sh
```
```

### 4. Handle Dependencies Gracefully

Check for required tools in server script:

```javascript
// mcp-server/index.js
import { exec } from 'child_process';

// Check for required tools
exec('which jq', (error) => {
  if (error) {
    console.error('Error: jq is required but not installed');
    process.exit(1);
  }
});
```

### 5. Use Appropriate Server Types

- **stdio**: Custom servers, local tools, full control
- **SSE**: Cloud services, OAuth, official integrations
- **HTTP**: REST APIs, token auth, stateless
- **ws**: Real-time, persistent connections

### 6. Test MCP Servers Independently

Before integrating with plugin:

```bash
# Test stdio server
node mcp-server/index.js

# Test with MCP inspector
npx @modelcontextprotocol/inspector node mcp-server/index.js
```

### 7. Provide Clear Error Messages

In MCP server code:

```javascript
if (!process.env.API_KEY) {
  throw new Error('API_KEY environment variable is required. Set it with: export API_KEY="your-key"');
}
```

### 8. Version Lock Dependencies

In package.json:

```json
{
  "dependencies": {
    "@modelcontextprotocol/sdk": "1.0.0"
  }
}
```

Not:
```json
{
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  }
}
```

## Debugging MCP Integration

### Check Server Status

```bash
# Test plugin with debug output
claude --debug --plugin-dir ./my-plugin

# Check MCP logs
tail -f ~/.claude/logs/mcp.log
```

### Validate Configuration

```bash
# Validate .mcp.json
cat .mcp.json | jq .

# Check environment variables
echo $CLAUDE_PLUGIN_ROOT
echo $API_TOKEN
```

### Test Server Directly

```bash
# Test stdio server
cd my-plugin/mcp-server
node index.js

# Check for errors
echo '{"method": "tools/list"}' | node index.js
```

### Common Issues

1. **Server not starting**: Check command path and args
2. **Authentication failing**: Verify environment variables set
3. **Tools not appearing**: Check server implements MCP protocol correctly
4. **Connection errors**: Verify URL and network connectivity

## MCP Tool Usage in Commands

Commands can invoke MCP tools like any other tool:

**commands/query.md**:
```yaml
---
description: Query the database using MCP tools
allowed-tools: ["Read", "mcp__plugin_database-tools_postgres__query"]
---

# Database Query Command

Query the database using the postgres MCP server.

Use the `mcp__plugin_database-tools_postgres__query` tool to execute queries.
```

## Reference

- [MCP Official Documentation](https://modelcontextprotocol.io)
- [MCP SDK](https://github.com/modelcontextprotocol/sdk)
- [MCP Servers](https://github.com/modelcontextprotocol/servers)

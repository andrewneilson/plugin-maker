# Command Advanced Features

Comprehensive patterns for creating powerful commands in Claude Code plugins.

## Command Basics

Commands are Markdown files with YAML frontmatter that Claude executes when invoked. They provide:
- Reusable workflows
- Consistent behavior
- Team standardization
- Quick access to complex tasks

## Critical: Commands are Instructions FOR Claude

Commands contain instructions that tell Claude what to do, not messages for users.

**Correct** (instructions for Claude):
```markdown
Review this code for security vulnerabilities including:
- SQL injection
- XSS attacks
- Authentication issues

Provide specific line numbers and severity ratings.
```

**Incorrect** (messages to user):
```markdown
This command will review your code for security issues.
You'll receive a report with vulnerability details.
```

The first tells Claude what to do. The second tells the user what will happen but doesn't instruct Claude.

## Basic Command Structure

```yaml
---
description: What this command does
argument-hint: [args]
allowed-tools: ["Read", "Write", "Bash"]
---

# Command Name

Instructions for Claude when this command is invoked.

The user's input is available as $ARGUMENTS.
```

## Advanced Features

### Bash Execution with ! Syntax

Execute bash commands inline within command instructions:

```markdown
# Git Status Command

Check the git repository status:

!git status

Then show uncommitted changes:

!git diff

Summarize the current state of the repository.
```

**Usage**:
- Inline bash with backtick-wrapped `!command`
- Results available to Claude immediately
- No need for explicit Bash tool calls

**Example - Branch info**:
```markdown
# Branch Info

Show current branch and recent commits:

!git branch --show-current
!git log --oneline -5

Explain the current development state.
```

### File References with @ Syntax

Include file contents directly in command:

```markdown
# Config Review

Review the current configuration:

@${CLAUDE_PLUGIN_ROOT}/config.json

Check for:
1. Missing required fields
2. Invalid values
3. Security issues
```

**Usage**:
- `@path/to/file` includes file contents
- Useful for templates and configs
- Supports `${CLAUDE_PLUGIN_ROOT}` variable

**Example - Template-based command**:
```markdown
# Generate Component

Create a new component based on template:

@${CLAUDE_PLUGIN_ROOT}/templates/component.tsx

Use this template to create $ARGUMENTS with:
- Props interface
- TypeScript types
- JSDoc comments
```

### Variables

#### $ARGUMENTS

User input passed to command:

```yaml
---
description: Greet user by name
argument-hint: [name]
---

# Greet

If $ARGUMENTS is provided, greet them by name.
Otherwise, ask for their name.
```

**Testing**:
```bash
/plugin:greet John
# $ARGUMENTS = "John"

/plugin:greet
# $ARGUMENTS = "" (empty)
```

#### ${CLAUDE_PLUGIN_ROOT}

Plugin directory path for portable paths:

```markdown
# Run Tests

Execute test suite using plugin's test runner:

!bash ${CLAUDE_PLUGIN_ROOT}/scripts/test.sh

Report test results.
```

**Why use it**:
- Works in any installation location
- No hardcoded paths
- Portable across users

#### ${CLAUDE_PROJECT_DIR}

Current project directory:

```markdown
# Project Summary

Analyze project structure:

!find ${CLAUDE_PROJECT_DIR} -type f -name "*.js" | wc -l

Summarize the project.
```

## Frontmatter Fields Reference

### description (Required)

Brief description shown in `/help`:

```yaml
description: Analyze code for security vulnerabilities
```

### argument-hint

Hint shown to user for expected arguments:

```yaml
argument-hint: [file-or-directory]
```

### allowed-tools

Restrict available tools (JSON array):

```yaml
allowed-tools: ["Read", "Grep", "Glob", "Bash"]
```

**Common combinations**:
```yaml
# Read-only
allowed-tools: ["Read", "Grep", "Glob"]

# Editor
allowed-tools: ["Read", "Write", "Edit", "Create"]

# Runner
allowed-tools: ["Bash"]

# Full access
allowed-tools: ["Read", "Write", "Edit", "Create", "Bash", "Grep", "Glob"]
```

### disable-model-invocation

If true, command only runs when explicitly invoked:

```yaml
disable-model-invocation: true
```

**Use when**:
- Command is expensive
- Command modifies external state
- Command requires explicit user action

## Command Patterns

### Configuration-Based Commands

Commands that read plugin configuration:

```markdown
---
description: Deploy using plugin configuration
---

# Deploy

Read deployment configuration:

@${CLAUDE_PLUGIN_ROOT}/deploy-config.yaml

Execute deployment:
1. Validate configuration
2. Run build
3. Deploy to configured target
4. Verify deployment

Report deployment status.
```

### Template-Based Commands

Commands that use templates to generate code:

```markdown
---
description: Create new API endpoint
argument-hint: [endpoint-name]
---

# Create Endpoint

Generate new API endpoint based on template:

@${CLAUDE_PLUGIN_ROOT}/templates/endpoint.ts

Create endpoint for $ARGUMENTS with:
- Request validation
- Error handling
- OpenAPI documentation
- Unit tests
```

### Multi-Script Commands

Commands that orchestrate multiple scripts:

```markdown
---
description: Run full test suite
allowed-tools: ["Bash", "Read"]
---

# Test Suite

Execute complete test pipeline:

!${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh
!${CLAUDE_PLUGIN_ROOT}/scripts/unit-tests.sh
!${CLAUDE_PLUGIN_ROOT}/scripts/integration-tests.sh
!${CLAUDE_PLUGIN_ROOT}/scripts/coverage.sh

Summarize test results and coverage.
```

### Interactive Commands

Commands that gather info from user:

```markdown
---
description: Interactive code review
---

# Code Review

Perform interactive code review:

1. Ask user which files to review
2. Read and analyze each file
3. For each issue found:
   - Show the issue
   - Ask if user wants fix applied
   - Apply fix if approved
4. Summarize changes made
```

### Agent Integration Commands

Commands that delegate to agents:

```markdown
---
description: Deep security analysis
allowed-tools: ["Task", "Read"]
---

# Security Analysis

Perform comprehensive security analysis:

1. Use task tool with security-analyzer agent
2. Provide agent with $ARGUMENTS or current directory
3. Collect agent findings
4. Summarize critical issues
5. Recommend remediation steps
```

### Validation Commands

Commands that check project state:

```markdown
---
description: Validate project readiness
allowed-tools: ["Read", "Bash", "Grep"]
---

# Validate Project

Check project is ready for release:

!npm run lint
!npm run test
!npm run build

Verify:
- [ ] All tests pass
- [ ] Build succeeds
- [ ] No linting errors
- [ ] Version bumped
- [ ] Changelog updated

Report validation status.
```

## Complete Command Examples

### Git Workflow Command

```yaml
---
description: Create feature branch with conventional naming
argument-hint: [feature-name]
---

# Feature Branch

Create and set up new feature branch:

!git checkout -b feature/$ARGUMENTS
!git push -u origin feature/$ARGUMENTS

Branch created: feature/$ARGUMENTS

Next steps:
1. Make your changes
2. Commit with conventional commits
3. Push to origin
4. Create pull request
```

### Database Migration Command

```yaml
---
description: Generate database migration
argument-hint: [migration-name]
allowed-tools: ["Bash", "Create", "Read"]
---

# Generate Migration

Create new database migration:

!timestamp=$(date +%Y%m%d%H%M%S)

Create migration files:
- migrations/${timestamp}_$ARGUMENTS.up.sql
- migrations/${timestamp}_$ARGUMENTS.down.sql

Use migration template:

@${CLAUDE_PLUGIN_ROOT}/templates/migration.sql

Ask user for:
- Tables to modify
- Columns to add/remove
- Indexes needed

Generate migration SQL.
```

### Documentation Generation Command

```yaml
---
description: Generate API documentation
allowed-tools: ["Read", "Grep", "Glob", "Create"]
---

# Generate Docs

Generate API documentation from code:

1. Find all API endpoints:
   !grep -r "app.get\|app.post\|app.put\|app.delete" src/

2. For each endpoint:
   - Extract route and method
   - Find handler function
   - Extract JSDoc comments
   - Generate OpenAPI spec

3. Create docs/api.md with:
   - Endpoint list
   - Request/response formats
   - Authentication requirements
   - Example requests
```

### Dependency Update Command

```yaml
---
description: Update project dependencies safely
allowed-tools: ["Bash", "Read"]
---

# Update Dependencies

Safely update project dependencies:

!npm outdated

For each outdated package:
1. Check changelog for breaking changes
2. Read migration guide if major version
3. Update package
4. Run tests
5. Commit if tests pass

Report update summary.
```

### Code Generation Command

```yaml
---
description: Generate CRUD operations
argument-hint: [entity-name]
allowed-tools: ["Create", "Read"]
---

# Generate CRUD

Generate complete CRUD operations for $ARGUMENTS:

Using templates:
@${CLAUDE_PLUGIN_ROOT}/templates/model.ts
@${CLAUDE_PLUGIN_ROOT}/templates/controller.ts
@${CLAUDE_PLUGIN_ROOT}/templates/routes.ts
@${CLAUDE_PLUGIN_ROOT}/templates/tests.ts

Create:
- src/models/$ARGUMENTS.ts (data model)
- src/controllers/$ARGUMENTS.controller.ts (business logic)
- src/routes/$ARGUMENTS.routes.ts (API routes)
- src/tests/$ARGUMENTS.test.ts (test suite)

Customize templates with entity-specific fields.
```

## Best Practices

### 1. Write Clear Instructions

```markdown
# Good
Review this code for security issues:
- SQL injection in database queries
- XSS in template rendering
- CSRF token validation

Provide file:line for each issue.

# Bad
Check the code.
```

### 2. Use Variables Appropriately

```markdown
# Good
If $ARGUMENTS is provided, analyze that file.
Otherwise, analyze current directory.

# Bad
Analyze the file.
```

### 3. Structure Multi-Step Commands

```markdown
# Good
1. Validate input
2. Run analysis
3. Generate report
4. Save results

# Bad
Do some stuff with the code.
```

### 4. Combine Features

```markdown
# Excellent - combines bash, file refs, and variables
Review configuration:

@${CLAUDE_PLUGIN_ROOT}/config-schema.json

Against current config:

!cat $ARGUMENTS

Validate all fields match schema.
```

### 5. Handle Edge Cases

```markdown
If $ARGUMENTS is empty:
  - Ask user which file to analyze
  - Default to current directory

If file not found:
  - Show available files
  - Ask user to select
```

### 6. Provide Context

```markdown
# Good
This command analyzes test coverage and identifies:
- Untested files
- Low coverage areas (< 80%)
- Missing edge case tests

# Okay
Analyze test coverage.
```

### 7. Use Appropriate Tool Restrictions

```yaml
# Analysis command - read only
allowed-tools: ["Read", "Grep", "Glob"]

# Deployment command - needs everything
allowed-tools: ["Read", "Bash"]

# Code generator - needs file creation
allowed-tools: ["Read", "Create", "Edit"]
```

## Command Locations

### Plugin Commands

```
my-plugin/
└── commands/
    ├── deploy.md       # /my-plugin:deploy
    ├── test.md         # /my-plugin:test
    └── help.md         # /my-plugin:help
```

**Namespace**: `/plugin-name:command-name`

### Project Commands

```
.claude/
└── commands/
    ├── review.md       # /review
    └── build.md        # /build
```

**Namespace**: `/command-name` (marked as "project")

### User Commands

```
~/.claude/
└── commands/
    ├── personal.md     # /personal
    └── utils.md        # /utils
```

**Namespace**: `/command-name` (marked as "user")

## Testing Commands

```bash
# Test command during development
claude --plugin-dir ./my-plugin

# Invoke command
/my-plugin:command-name arg1 arg2

# Debug command
claude --debug --plugin-dir ./my-plugin
```

## Common Patterns

### "Ask Then Act"

```markdown
1. Ask user for requirements
2. Validate requirements
3. Execute action
4. Report results
```

### "Validate Then Proceed"

```markdown
1. Check prerequisites
2. If valid: proceed with action
3. If invalid: explain issues and stop
```

### "Template Then Customize"

```markdown
1. Load template file
2. Ask user for customizations
3. Apply customizations
4. Create final file
```

### "Batch Process"

```markdown
1. Find all matching files
2. For each file:
   - Perform operation
   - Collect results
3. Summarize all results
```

## Debugging Commands

### Check Frontmatter Syntax

```bash
# Verify YAML parsing
head -20 commands/my-command.md
```

### Test Variable Expansion

```markdown
# Debug command
Echo variables:

$ARGUMENTS
${CLAUDE_PLUGIN_ROOT}
${CLAUDE_PROJECT_DIR}
```

### Verify Bash Execution

```markdown
# Test bash syntax
!echo "Testing bash execution"
!pwd
```

### Check File References

```markdown
# Test file loading
@${CLAUDE_PLUGIN_ROOT}/test-file.txt
```

## Reference

- See EXAMPLES.md for complete command examples
- See TEMPLATES.md for command templates
- See VALIDATION.md for command validation checklist

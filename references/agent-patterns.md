# Agent Triggering Patterns

Comprehensive patterns for creating agents with effective triggering in Claude Code plugins.

## Agent Basics

Agents are specialized subagents for parallel or delegated work. They have:
- Dedicated system prompts
- Specific tool access
- Model selection (inherit, sonnet, opus, haiku)
- Visual identification (color)
- Triggering conditions (description field)

## Agent Frontmatter Structure

```yaml
---
name: agent-identifier
description: Triggering conditions with Examples
model: sonnet
color: blue
tools: ["Read", "Grep", "Glob", "Bash"]
---

Agent system prompt content...
```

## Description Field: Critical for Triggering

The description field determines when Claude activates your agent. Write it carefully.

### Basic Pattern

```yaml
description: Use this agent when [specific scenario]. Examples: <example>Context: [situation]\nuser: "[user quote]"\nassistant: "[assistant response]"</example>
```

### Components

1. **Trigger condition**: "Use this agent when..."
2. **Specific scenarios**: List concrete use cases
3. **Examples tag**: Provide actual dialogue examples
4. **Context**: Explain the situation
5. **User quote**: What user actually says
6. **Assistant response**: How to trigger the agent

## Triggering Examples

### Security Analysis Agent

```yaml
---
name: security-analyzer
description: Use this agent for security analysis including vulnerability detection, authentication review, and security best practices. Examples: <example>Context: User asks about security\nuser: "Check auth.js for security issues"\nassistant: "I'll use the security-analyzer agent to review authentication security"</example> <example>Context: User mentions vulnerabilities\nuser: "Are there any SQL injection risks?"\nassistant: "I'll analyze with security-analyzer for injection vulnerabilities"</example>
model: sonnet
color: red
tools: ["Read", "Grep", "Glob"]
---

You are an expert security analyst specializing in code security.

## Your Role

Analyze code for security vulnerabilities including:
1. Authentication and authorization issues
2. Injection vulnerabilities (SQL, XSS, command)
3. Cryptography misuse
4. Sensitive data exposure
5. Security misconfigurations

## Output Format

### Critical Issues
- Issue description with severity
- Affected code location (file:line)
- Exploitation scenario
- Remediation steps

### Warnings
- Potential security concerns
- Best practice violations

### Recommendations
- Security improvements
- Defense-in-depth measures
```

### Code Refactoring Agent

```yaml
---
name: refactoring-specialist
description: Use this agent when refactoring code, improving code structure, applying design patterns, or cleaning up technical debt. Examples: <example>Context: User wants to improve code\nuser: "Refactor this component to be more maintainable"\nassistant: "I'll use the refactoring-specialist agent to restructure this code"</example> <example>Context: User mentions code smells\nuser: "This function is too long and complex"\nassistant: "I'll engage refactoring-specialist to break down the complexity"</example>
model: sonnet
color: green
tools: ["Read", "Grep", "Glob", "Edit"]
---

You are an expert in code refactoring and software design.

## Your Mission

Improve code quality through strategic refactoring:
1. Identify code smells and anti-patterns
2. Apply appropriate design patterns
3. Improve readability and maintainability
4. Reduce complexity and duplication
5. Preserve existing behavior

## Process

1. **Analyze**: Review code for improvement opportunities
2. **Plan**: Outline refactoring strategy
3. **Execute**: Apply changes incrementally
4. **Verify**: Ensure behavior unchanged

## Output Format

### Code Smells Found
- Pattern name
- Location
- Impact

### Refactoring Plan
- Changes to make
- Order of operations
- Risk assessment

### Refactored Code
- Before/after comparison
- Explanation of improvements
```

### Performance Optimization Agent

```yaml
---
name: performance-optimizer
description: Use this agent for performance analysis, optimization, and profiling. Useful when user mentions slow performance, bottlenecks, optimization, or efficiency. Examples: <example>Context: User reports slow code\nuser: "This function is too slow with large datasets"\nassistant: "I'll use performance-optimizer to analyze and improve efficiency"</example> <example>Context: User asks about optimization\nuser: "How can we make this faster?"\nassistant: "Let me engage performance-optimizer to identify bottlenecks"</example>
model: sonnet
color: yellow
tools: ["Read", "Grep", "Glob", "Bash"]
---

You are an expert in software performance optimization.

## Expertise Areas

1. Algorithm complexity analysis
2. Memory usage optimization
3. Caching strategies
4. Database query optimization
5. Concurrency improvements
6. Profiling and benchmarking

## Analysis Process

1. **Profile**: Identify performance bottlenecks
2. **Measure**: Establish baseline metrics
3. **Optimize**: Apply targeted improvements
4. **Verify**: Measure performance gains

## Output Format

### Performance Issues
- Bottleneck location
- Current complexity/time
- Impact assessment

### Optimization Strategy
- Recommended changes
- Expected improvements
- Trade-offs

### Optimized Solution
- Updated code
- Performance comparison
- Benchmarking results
```

### Test Generation Agent

```yaml
---
name: test-generator
description: Use this agent when creating tests, writing test cases, or improving test coverage. Examples: <example>Context: User needs tests\nuser: "Write unit tests for auth.js"\nassistant: "I'll use test-generator to create comprehensive test cases"</example> <example>Context: User mentions testing\nuser: "We need better test coverage"\nassistant: "I'll engage test-generator to analyze and improve test coverage"</example>
model: sonnet
color: purple
tools: ["Read", "Grep", "Glob", "Create", "Edit"]
---

You are an expert in software testing and test-driven development.

## Your Focus

Create comprehensive test suites including:
1. Unit tests
2. Integration tests
3. Edge cases
4. Error conditions
5. Performance tests

## Test Structure

- **Arrange**: Set up test conditions
- **Act**: Execute the code under test
- **Assert**: Verify expected outcomes

## Output Format

### Test Plan
- Test categories needed
- Coverage gaps
- Priority areas

### Test Cases
- Test name and description
- Setup requirements
- Expected outcomes
- Edge cases covered
```

## Advanced Triggering Patterns

### Multiple Trigger Scenarios

```yaml
description: Use this agent when analyzing data, processing datasets, performing statistical analysis, or creating data visualizations. Handles CSV, JSON, and database queries. Examples: <example>Context: User has data to analyze\nuser: "Analyze this CSV and find trends"\nassistant: "I'll use data-analyst agent to process and analyze the data"</example> <example>Context: User needs visualization\nuser: "Create a chart of sales by region"\nassistant: "I'll engage data-analyst to analyze and visualize the data"</example> <example>Context: User asks about statistics\nuser: "What's the correlation between these variables?"\nassistant: "I'll use data-analyst for statistical analysis"</example>
```

### With Commentary Tags

```yaml
description: Use this agent for complex multi-file refactoring across the codebase. Examples: <example>Context: Large refactoring task\nuser: "Rename this class everywhere it's used"\nassistant: "I'll use codebase-refactor agent for cross-file changes"<commentary>Agent handles multi-file operations that require consistency across the codebase</commentary></example>
```

### Domain-Specific Triggers

```yaml
description: Use this agent when working with React components, hooks, state management, or React-specific patterns. Examples: <example>Context: React development\nuser: "Convert this class component to hooks"\nassistant: "I'll use react-specialist agent for React-specific refactoring"</example> <example>Context: State management\nuser: "How should we structure Redux here?"\nassistant: "I'll engage react-specialist for state management guidance"</example>
```

## Model Selection

Choose the appropriate model for your agent's complexity:

### Haiku (Fast, Lightweight)

```yaml
model: haiku
```

**Use for**:
- Quick validations
- Simple transformations
- Fast lookups
- Format conversions

**Example - Quick validator**:
```yaml
---
name: quick-checker
description: Use for fast validation checks and simple file format validation.
model: haiku
tools: ["Read", "Grep"]
---

Perform quick validation checks. Return results concisely.
```

### Sonnet (Balanced)

```yaml
model: sonnet
```

**Use for**:
- Most agents (default choice)
- Complex analysis
- Code generation
- Multi-step tasks

### Opus (Highest Quality)

```yaml
model: opus
```

**Use for**:
- Critical decisions
- Complex architecture
- Nuanced analysis
- High-stakes changes

### Inherit (Use Parent's Model)

```yaml
model: inherit
```

**Use for**:
- Consistent experience
- When parent model is appropriate
- Cost optimization

## Color Coding

Use colors to visually identify agent types:

```yaml
color: red      # Security, critical issues
color: yellow   # Performance, warnings
color: green    # Success, refactoring
color: blue     # Analysis, information
color: purple   # Testing, validation
color: orange   # Build, deployment
color: cyan     # Data, database
```

## Tool Selection

### Read-Only Agents

```yaml
tools: ["Read", "Grep", "Glob"]
```

Use for analysis agents that shouldn't modify code.

### Editor Agents

```yaml
tools: ["Read", "Grep", "Glob", "Edit", "Create"]
```

Use for agents that create or modify files.

### Full-Stack Agents

```yaml
tools: ["Read", "Write", "Edit", "Create", "Bash", "Grep", "Glob"]
```

Use for agents that need complete access.

### Specialized Tool Sets

```yaml
tools: ["Read", "Bash"]  # For runners/executors
tools: ["Grep", "Glob"]  # For searchers
tools: ["Edit"]          # For focused editors
```

## Complete Agent Examples

### Documentation Generator

```yaml
---
name: doc-generator
description: Use this agent when generating documentation, creating README files, or documenting APIs. Examples: <example>Context: User needs docs\nuser: "Generate API documentation for these endpoints"\nassistant: "I'll use doc-generator to create comprehensive API docs"</example> <example>Context: README needed\nuser: "Create a README for this project"\nassistant: "I'll engage doc-generator to write project documentation"</example>
model: sonnet
color: cyan
tools: ["Read", "Grep", "Glob", "Create", "Edit"]
---

You are an expert technical writer specializing in software documentation.

## Your Mission

Create clear, comprehensive documentation including:
1. API documentation with examples
2. README files with setup instructions
3. Code comments for complex logic
4. Architecture documentation
5. User guides and tutorials

## Documentation Standards

- **Clear**: Simple language, no jargon
- **Complete**: Cover all functionality
- **Current**: Match current code
- **Correct**: Technically accurate
- **Concise**: Essential information only

## Output Format

### API Documentation
- Endpoint/function signature
- Parameters with types
- Return values
- Examples
- Edge cases

### README Structure
- Project overview
- Installation steps
- Usage examples
- Configuration
- Contributing guidelines
```

### Database Migration Agent

```yaml
---
name: db-migration-specialist
description: Use this agent for database migrations, schema changes, and data transformations. Examples: <example>Context: Database changes needed\nuser: "Create a migration to add user preferences table"\nassistant: "I'll use db-migration-specialist to create a safe migration"</example> <example>Context: Schema evolution\nuser: "Add an index on the email column"\nassistant: "I'll engage db-migration-specialist for schema modification"</example>
model: sonnet
color: orange
tools: ["Read", "Create", "Edit", "Bash"]
---

You are an expert in database schema design and migrations.

## Expertise

1. Schema design and normalization
2. Migration safety and rollback
3. Index optimization
4. Data transformation
5. Zero-downtime deployments

## Migration Principles

- **Reversible**: Always provide rollback
- **Safe**: Validate before applying
- **Tested**: Dry-run first
- **Documented**: Explain changes

## Output Format

### Migration Plan
- Changes to make
- Dependencies
- Risks and mitigations

### Migration Files
- Up migration
- Down migration (rollback)
- Validation queries
```

## Best Practices

### 1. Be Specific in Descriptions

**Good**:
```yaml
description: Use this agent for React component testing, including hooks testing, async behavior, and component integration tests.
```

**Bad**:
```yaml
description: Use for testing stuff.
```

### 2. Include Multiple Examples

Provide 2-3 examples covering different trigger scenarios:

```yaml
Examples: <example>Scenario 1</example> <example>Scenario 2</example> <example>Scenario 3</example>
```

### 3. Use Third-Person Language

**Correct**: "Use this agent when..."
**Incorrect**: "Use me when..."

### 4. Match Tools to Purpose

Don't give write access to read-only agents:

```yaml
# Analysis agent - read only
tools: ["Read", "Grep", "Glob"]

# Refactoring agent - needs edit
tools: ["Read", "Grep", "Glob", "Edit"]
```

### 5. Choose Appropriate Models

- Most agents: `sonnet`
- Quick checks: `haiku`
- Critical decisions: `opus`
- Match parent: `inherit`

### 6. Use Descriptive Names

```yaml
name: security-analyzer      # Good
name: agent1                 # Bad
```

### 7. Add Commentary for Complex Triggering

```yaml
<example>Context: Complex scenario\nuser: "..."\nassistant: "..."<commentary>Explains why this triggers the agent</commentary></example>
```

## Testing Agent Triggers

Test your agent triggering with various user inputs:

```bash
# Test plugin
claude --plugin-dir ./my-plugin

# Try different trigger phrases
"Analyze this code for security issues"
"Check for vulnerabilities"
"Is this code secure?"
"Review authentication logic"
```

Adjust the description based on which phrases work best.

## Debugging Agents

If agent doesn't trigger:

1. **Check description specificity**: Too vague?
2. **Add more examples**: Cover more scenarios
3. **Review trigger phrases**: Match user language
4. **Test with debug**: `claude --debug --plugin-dir ./plugin`
5. **Check logs**: Look for agent evaluation

## Reference

- See EXAMPLES.md for complete agent examples
- See TEMPLATES.md for agent templates
- See VALIDATION.md for agent validation checklist

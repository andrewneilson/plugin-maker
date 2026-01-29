# Copilot Instructions for plugin-maker

This repository contains a Claude Code skill that teaches users how to create plugins.

## Repository Structure

- `SKILL.md` - Main skill file with plugin creation instructions
- `references/EXAMPLES.md` - Complete plugin examples
- `references/TEMPLATES.md` - Copy-paste templates
- `references/VALIDATION.md` - Validation checklists
- `references/MISTAKES.md` - Common mistakes to avoid

## Coding Guidelines

1. **Markdown Style**
   - Use consistent heading levels
   - Include code blocks with language hints (```yaml, ```json, etc.)
   - Keep tables aligned
   - No trailing spaces on any line

2. **Content Guidelines**
   - Focus on practical, actionable instructions
   - Include working examples that can be copy-pasted
   - Warn about common pitfalls
   - Cross-reference related sections with relative paths

3. **Upstream Sync Tasks**
   When syncing from claude-plugins-official:
   - We are a standalone skill, NOT a full plugin
   - Preserve our simpler structure (SKILL.md + references/)
   - Only add content that improves user experience
   - Keep SKILL.md under 500 lines
   - Move detailed content to references/ directory

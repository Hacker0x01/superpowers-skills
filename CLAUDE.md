# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code marketplace that distributes modular skills plugins organized by category. It's a fork of [superpowers](https://github.com/obra/superpowers) maintained at [Hacker0x01/claude-power-user](https://github.com/Hacker0x01/claude-power-user) and restructured into independent plugin packages.

## Repository Structure

```
claude-power-user/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration with plugin catalog
├── plugins/                      # Individual plugin packages
│   ├── collaboration/
│   ├── testing/
│   ├── debugging/
│   ├── meta/
│   ├── problem-solving/
│   ├── research/
│   └── architecture/
└── skills/                       # Original reference directory (legacy)
```

Each plugin follows this structure:
```
plugins/[category]/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata (name, version, author)
└── skills/
    └── [skill-name]/
        └── SKILL.md             # Skill instructions for Claude
```

## Architecture

### Marketplace System

The marketplace is defined in `.claude-plugin/marketplace.json` which:
- Lists all available plugins with metadata (name, description, version, category, keywords)
- Uses `"source": "local"` references pointing to `plugins/[category]` directories
- Allows users to selectively install skill categories: `/plugin install [category]-skills@claude-power-user`
- Each plugin entry has `"strict": true` to enforce exact skill matching

### Skills System

Skills are specialized instructions that guide Claude Code through specific workflows:
- Each skill is a SKILL.md file in a skill directory
- Skills provide proven patterns, techniques, and systematic approaches
- Skills are discovered and invoked by Claude Code's native skills system
- The remembering-conversations skill includes a Node.js tool with vector search (sqlite-vec, @xenova/transformers)

### Plugin Development Pattern

Plugins are independent, focused on a single category:
- **collaboration-skills**: brainstorming, planning, code review, parallel agents, git workflows (10 skills)
- **testing-skills**: TDD, async patterns, anti-patterns (3 skills)
- **debugging-skills**: systematic debugging, root cause tracing, verification (4 skills)
- **problem-solving-skills**: thinking techniques, simplification, pattern recognition (6 skills)
- **meta-skills**: writing skills, testing skills, sharing contributions (5 skills)
- **research-skills**: knowledge investigation, tracing lineages (1 skill)
- **architecture-skills**: design patterns, productive tensions (1 skill)

## Development Commands

### Testing Remembering Conversations Tool

```bash
cd plugins/collaboration/skills/remembering-conversations/tool
npm install
npm test           # Run tests once
npm run test:watch # Run tests in watch mode
npm run index      # Index conversations
npm run search     # Search indexed conversations
```

The tool includes comprehensive test coverage via vitest and test deployment scripts.

## Philosophy

Skills embody these principles:
- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success
- **Domain over implementation** - Work at problem level, not solution level

## Making Changes

### Adding a New Skill to Existing Plugin

1. Navigate to plugin: `plugins/[category]/skills/`
2. Create skill directory with SKILL.md file
3. Update plugin version in `plugins/[category]/.claude-plugin/plugin.json`
4. Test skill can be discovered and invoked
5. Update marketplace version in `.claude-plugin/marketplace.json` if needed

### Creating a New Plugin Category

1. Create directory: `plugins/[new-category]/`
2. Add `.claude-plugin/plugin.json` with metadata
3. Create `skills/` directory with initial SKILL.md files
4. Add plugin entry to `.claude-plugin/marketplace.json`
5. Test installation: `/plugin install [new-category]-skills@claude-power-user`

### Modifying the Marketplace

Edit `.claude-plugin/marketplace.json` to:
- Update plugin descriptions or versions
- Change plugin categories or keywords
- Add/remove plugins from catalog
- Modify marketplace metadata

## Version Management

- Marketplace version: currently 2.0.0 (in `.claude-plugin/marketplace.json`)
- Individual plugin versions: 1.0.0 (in each `plugins/[category]/.claude-plugin/plugin.json`)
- Increment versions when making significant changes to ensure users get updates

## Legacy Structure

The `skills/` directory at the root is the original structure before modularization. It contains:
- Same skills as `plugins/` but in monolithic structure
- `REQUESTS.md` - Template for documenting skill requests
- Kept as reference, but plugins are the active distribution method

## Testing Changes

Since there's no build system:
1. Make changes to SKILL.md files or plugin metadata
2. If testing locally, Claude Code reads directly from plugin directories
3. Verify skill content by invoking skills in a conversation
4. Confirm plugin installation works: `/plugin install [plugin-name]@claude-power-user`

## Contributing

Skills development follows TDD principles:
1. Identify gap or need (document in skills/REQUESTS.md)
2. Design skill structure and instructions
3. Test skill with Claude Code in real scenarios
4. Iterate on skill content based on effectiveness
5. Submit PR with skill changes and version bumps

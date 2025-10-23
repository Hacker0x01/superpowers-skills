# Claude Power User Marketplace

A comprehensive Claude Code skills marketplace with modular plugins organized by skill category.

## Overview

This marketplace provides a collection of specialized skills plugins for Claude Code, organized into focused categories for easy installation and management.

## Installation

### Add the Marketplace

```bash
/plugin marketplace add Hacker0x01/claude-power-user
```

### Install by Role Pack (Recommended)

Choose the pack that matches your role for a curated selection of skills:

#### Universal Pack (All Users)
Essential skills for collaboration, problem-solving, and research - perfect for non-technical users, managers, and researchers.

```bash
/plugin install collaboration-skills@claude-power-user && /plugin install problem-solving-skills@claude-power-user && /plugin install research-skills@claude-power-user
```

**Includes:** collaboration-skills, problem-solving-skills, research-skills

#### Developer Pack
Developer-focused skills for testing, debugging, and architecture - ideal for engineers and tech leads.

```bash
/plugin install testing-skills@claude-power-user && /plugin install debugging-skills@claude-power-user && /plugin install architecture-skills@claude-power-user
```

**Includes:** testing-skills, debugging-skills, architecture-skills

#### Power User Pack
Meta skills for customizing Claude Code and creating your own skills - for admins and power users.

```bash
/plugin install meta-skills@claude-power-user
```

**Includes:** meta-skills

#### Complete Pack (Everything)
Install all available skills for the full claude-power-user experience.

```bash
/plugin install collaboration-skills@claude-power-user && /plugin install testing-skills@claude-power-user && /plugin install debugging-skills@claude-power-user && /plugin install meta-skills@claude-power-user && /plugin install problem-solving-skills@claude-power-user && /plugin install research-skills@claude-power-user && /plugin install architecture-skills@claude-power-user
```

**Includes:** All 7 skill plugins

### Install Individual Plugins

You can also install only the specific skill categories you need:

```bash
# Collaboration skills
/plugin install collaboration-skills@claude-power-user

# Testing skills
/plugin install testing-skills@claude-power-user

# Debugging skills
/plugin install debugging-skills@claude-power-user

# Problem-solving skills
/plugin install problem-solving-skills@claude-power-user

# Meta skills (for creating and managing skills)
/plugin install meta-skills@claude-power-user

# Research skills
/plugin install research-skills@claude-power-user

# Architecture skills
/plugin install architecture-skills@claude-power-user
```

## Available Plugins

### Collaboration Skills
**Category:** Productivity
**Skills:** Brainstorming, planning, code review, parallel agents, git workflows, remembering conversations

Key skills:
- `brainstorming` - Interactive design refinement using Socratic method
- `writing-plans` - Create detailed implementation plans
- `executing-plans` - Execute plans in batches with review checkpoints
- `requesting-code-review` - Pre-review checklist
- `receiving-code-review` - Responding to feedback
- `dispatching-parallel-agents` - Concurrent subagent workflows
- `using-git-worktrees` - Parallel development branches
- `finishing-a-development-branch` - Clean branch completion
- `subagent-driven-development` - Delegation patterns
- `remembering-conversations` - Search past work

### Testing Skills
**Category:** Development
**Skills:** TDD, async patterns, testing anti-patterns

Key skills:
- `test-driven-development` - RED-GREEN-REFACTOR cycle
- `condition-based-waiting` - Async test patterns
- `testing-anti-patterns` - Common pitfalls to avoid

### Debugging Skills
**Category:** Development
**Skills:** Systematic debugging, root cause tracing, verification, defense-in-depth

Key skills:
- `systematic-debugging` - 4-phase root cause process
- `root-cause-tracing` - Find the real problem
- `verification-before-completion` - Ensure it's actually fixed
- `defense-in-depth` - Multiple validation layers

### Problem-Solving Skills
**Category:** Productivity
**Skills:** Thinking techniques, simplification, pattern recognition, inversion exercises

Key skills:
- `collision-zone-thinking` - Find interaction points
- `simplification-cascades` - Reduce complexity systematically
- `meta-pattern-recognition` - Identify patterns across domains
- `inversion-exercise` - Think backwards to solve forward
- `scale-game` - Explore problem at different scales
- `when-stuck` - Strategies for breaking through blocks

### Meta Skills
**Category:** Meta
**Skills:** Writing skills, testing skills, sharing contributions, maintaining skill repositories

Key skills:
- `writing-skills` - TDD for documentation, create new skills
- `testing-skills-with-subagents` - Validate skill quality
- `sharing-skills` - Contribute skills back via branch and PR
- `pulling-updates-from-skills-repository` - Sync with upstream
- `gardening-skills-wiki` - Maintain and improve skills

### Research Skills
**Category:** Research
**Skills:** Knowledge investigation, tracing lineages, information gathering

Key skills:
- `tracing-knowledge-lineages` - Follow information sources and evolution

### Architecture Skills
**Category:** Architecture
**Skills:** Design patterns, productive tensions, system thinking

Key skills:
- `preserving-productive-tensions` - Balance competing concerns in design

## Philosophy

These skills embody proven development principles:
- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success
- **Domain over implementation** - Work at problem level, not solution level

## Repository Structure

```
claude-power-user/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace configuration
├── plugins/
│   ├── collaboration/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin metadata
│   │   └── skills/               # Collaboration skills
│   ├── testing/                  # Testing skills plugin
│   ├── debugging/                # Debugging skills plugin
│   ├── meta/                     # Meta skills plugin
│   ├── problem-solving/          # Problem-solving skills plugin
│   ├── research/                 # Research skills plugin
│   └── architecture/             # Architecture skills plugin
└── skills/                       # Original skills directory (reference)
```

## Contributing

To contribute improvements to skills:

1. Fork this repository
2. Make your changes to skills in the appropriate plugin directory
3. Test your changes
4. Submit a pull request

## License

MIT License - see LICENSE file for details

## Credits

Skills originally derived from the [superpowers](https://github.com/obra/superpowers) project by Jesse Vincent and forked from [Hacker0x01/claude-power-user](https://github.com/Hacker0x01/claude-power-user).

---
name: Vibe Kanban MCP
description: Multi-agent orchestration and project management using Vibe Kanban MCP tools for tracking complex workflows
when_to_use: when using any Vibe Kanban MCP tool (mcp__vibe_kanbam__*) for project management, multi-agent orchestration, or tracking work across sessions
version: 1.0.0
languages: all
dependencies: Vibe Kanban MCP server configured
---

# Vibe Kanban MCP

## Overview

**Vibe Kanban is an orchestration and project management tool that runs locally via MCP.**

It provides persistent task tracking across sessions, multi-agent orchestration, and project management capabilities. Think of it as a local Kanban board combined with agent task assignment.

**Core principle:** Use Vibe Kanban when you need persistent tracking, multi-agent coordination, or structured project management across multiple work sessions.

## When to Use

**Use Vibe Kanban when:**
- Coordinating multiple agents working on independent tasks
- Tracking work that spans multiple sessions
- Managing complex projects with many components
- Need to assign specific tasks to specific agents (with executors/branches)
- Want centralized view of project progress
- Planning and architecting larger features

**Use TodoWrite instead when:**
- Single session, simple task list
- You're the only agent working
- Tasks are tightly coupled and sequential
- Just need progress tracking within current conversation

**Use neither when:**
- Single straightforward task
- Investigation/debugging that doesn't need tracking
- Direct work is more efficient than orchestration overhead

## Quick Reference

### Core Workflow

```
1. list_projects      → Find or verify project_id
2. create_task       → Create tasks (requires project_id!)
3. list_tasks        → View all tasks in project
4. start_task_attempt → Launch agent to work on specific task
5. get_task          → Check details/status of specific task
6. update_task       → Change title, description, or status
7. delete_task       → Remove task from project
```

### Tool Parameters

| Tool | Required | Optional | Returns |
|------|----------|----------|---------|
| `list_projects` | none | none | All projects with IDs |
| `create_task` | project_id, title | description | Task with UUID |
| `list_tasks` | project_id | status, limit | Tasks in project |
| `get_task` | task_id | none | Task details |
| `update_task` | task_id | title, description, status | Updated task |
| `delete_task` | task_id | none | Confirmation |
| `start_task_attempt` | task_id, executor, base_branch | variant | Attempt ID + launch |

### Task Statuses

- `todo` - Not started
- `inprogress` - Actively being worked on
- `inreview` - Done, needs review
- `done` - Completed
- `cancelled` - No longer needed

### Available Executors

- `CLAUDE_CODE` - Claude Code (most common)
- `CODEX` - OpenAI Codex
- `GEMINI` - Google Gemini
- `CURSOR` - Cursor AI
- `OPENCODE` - Open source alternative

## Common Workflows

### Creating a New Project with Tasks

```typescript
// 1. Check existing projects first
const projects = await list_projects();

// 2. Create tasks with proper project_id
const tasks = await Promise.all([
  create_task({
    project_id: "4e8f1a12-5db8-4057-ba74-a20d7127d110",
    title: "Database schema migrations",
    description: "Add tables for new payment system"
  }),
  create_task({
    project_id: "4e8f1a12-5db8-4057-ba74-a20d7127d110",
    title: "API endpoint creation",
    description: "Create REST endpoints for payment processing"
  }),
  create_task({
    project_id: "4e8f1a12-5db8-4057-ba74-a20d7127d110",
    title: "Frontend UI components"
  })
]);
```

### Orchestrating Multiple Agents

```typescript
// 1. Create tasks for each work stream
const debugTask = await create_task({
  project_id: projectId,
  title: "Debug payment processing error"
});

const hotfixTask = await create_task({
  project_id: projectId,
  title: "Implement hotfix for race condition"
});

// 2. Start agents on specific tasks
await start_task_attempt({
  task_id: debugTask.id,
  executor: "CLAUDE_CODE",
  base_branch: "develop"
});

await start_task_attempt({
  task_id: hotfixTask.id,
  executor: "CLAUDE_CODE",
  base_branch: "bugfix/payments"
});

// 3. Monitor progress
const tasks = await list_tasks({
  project_id: projectId,
  status: "inprogress"
});
```

### Checking Task Status

```typescript
// Get specific task details
const task = await get_task({ task_id: "abc-123-..." });

// List all in-progress tasks
const active = await list_tasks({
  project_id: projectId,
  status: "inprogress"
});

// Update task status after completion
await update_task({
  task_id: taskId,
  status: "done"
});
```

## Critical Rules

### ALWAYS Call list_projects First

```typescript
// ✅ CORRECT: Get project_id before creating tasks
const projects = await list_projects();
const projectId = projects.find(p => p.path === "/path/to/repo")?.id;

await create_task({ project_id: projectId, title: "..." });

// ❌ WRONG: Guessing or skipping project_id
await create_task({ project_id: "some-guess", title: "..." });
```

**Why:** `project_id` is mandatory for create_task. The tool will error without it. Always list projects first to get valid UUIDs.

### Use start_task_attempt, Not Regular Task Tool

```typescript
// ✅ CORRECT: Kanban-aware agent dispatch
await start_task_attempt({
  task_id: "task-uuid-here",
  executor: "CLAUDE_CODE",
  base_branch: "develop"
});

// ❌ WRONG: Regular Task tool doesn't update Kanban
await Task({
  subagent_type: "general-purpose",
  description: "Work on task",
  prompt: "..."
});
```

**Why:** `start_task_attempt` creates the task attempt record in Kanban and tracks which agent is working on what. Regular Task tool bypasses Kanban tracking.

## Common Mistakes

### ❌ Skipping list_projects

```typescript
// Creates error - project_id is required
await create_task({
  project_id: undefined, // FAILS
  title: "My task"
});
```

**Fix:** Always call `list_projects()` first to get valid project_id.

### ❌ Using TodoWrite for Multi-Agent Work

```typescript
// TodoWrite doesn't support:
// - Multiple agents working independently
// - Task assignment to specific executors
// - Persistent tracking across sessions
```

**Fix:** Use Vibe Kanban when you need orchestration features.

### ❌ Using Regular Task Tool for Kanban Tasks

```typescript
// This doesn't update Kanban tracking
await Task({ description: "Work on payment task", ... });
```

**Fix:** Use `start_task_attempt` with the task_id from Kanban.

### ❌ Forgetting to Check project_id Format

Project IDs are UUIDs, not names:
- ✅ `"4e8f1a12-5db8-4057-ba74-a20d7127d110"`
- ❌ `"my-project"` or `"core"`

**Fix:** Use the ID field from list_projects results, not the name or path.

## Decision Framework

```
Need to track work?
  │
  ├─ Single session only?
  │  └─ Use TodoWrite
  │
  ├─ Multiple agents in parallel?
  │  └─ Use Vibe Kanban
  │
  ├─ Work spans multiple sessions?
  │  └─ Use Vibe Kanban
  │
  ├─ Need specific executor/branch per task?
  │  └─ Use Vibe Kanban
  │
  └─ Simple investigation work?
     └─ Use neither (just work)
```

## Best Practices

1. **Create tasks before starting agents** - Don't skip straight to `start_task_attempt` without creating the task first

2. **Use descriptive titles** - "Fix payment bug" not "Bug fix"

3. **Add descriptions for complex tasks** - Help future agents understand context

4. **Check task status regularly** - Use `list_tasks` to monitor progress

5. **Update status appropriately** - Move tasks through todo → inprogress → inreview → done

6. **Clean up completed tasks** - Archive or delete done tasks to reduce clutter

7. **Use status filtering** - `list_tasks({ status: "inprogress" })` to focus on active work

## Real-World Example

**Scenario:** Implementing a new payment processing system

```typescript
// 1. Verify project
const projects = await list_projects();
const coreProject = projects.find(p => p.name === "core");

// 2. Create task breakdown
const tasks = await Promise.all([
  create_task({
    project_id: coreProject.id,
    title: "Database schema migrations",
    description: "Add payment_transactions and payment_methods tables"
  }),
  create_task({
    project_id: coreProject.id,
    title: "Payment service API",
    description: "Create REST endpoints: POST /payments, GET /payments/:id"
  }),
  create_task({
    project_id: coreProject.id,
    title: "Frontend payment form",
    description: "React component for credit card input with validation"
  }),
  create_task({
    project_id: coreProject.id,
    title: "Background reconciliation job",
    description: "Sidekiq job to reconcile payments with Stripe daily"
  }),
  create_task({
    project_id: coreProject.id,
    title: "Integration tests",
    description: "End-to-end test for complete payment flow"
  })
]);

// 3. Start agents on independent work streams
await start_task_attempt({
  task_id: tasks[0].id, // Database
  executor: "CLAUDE_CODE",
  base_branch: "develop"
});

await start_task_attempt({
  task_id: tasks[1].id, // API
  executor: "CLAUDE_CODE",
  base_branch: "develop"
});

// 4. Monitor progress
const inProgress = await list_tasks({
  project_id: coreProject.id,
  status: "inprogress"
});

console.log(`${inProgress.length} tasks currently in progress`);
```

## Troubleshooting

**Error: "project_id is required"**
- Call `list_projects()` first to get valid UUID

**Error: "task_id not found"**
- Verify task exists with `list_tasks()`
- Check you're using the task UUID, not the title

**Can't find task attempt after starting**
- Task attempts are tracked internally
- Use `get_task()` to see task status changes

**Agent not updating task status**
- Agent must explicitly call `update_task()`
- Consider adding status update to agent instructions

## See Also

- skills/collaboration/subagent-driven-development - For using agents effectively
- skills/collaboration/dispatching-parallel-agents - For parallel work patterns
- skills/collaboration/writing-plans - For planning before execution

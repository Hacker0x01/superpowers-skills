# Vibe Kanban Baseline Test Scenarios

## Scenario 1: Large Feature with Multi-Agent Orchestration

**Context:** User wants to implement a complex feature that requires multiple agents working on independent subtasks.

**Pressure Applied:**
- Time: "I need this done quickly"
- Authority: "Can't you just use TodoWrite for this?"
- Complexity: 5+ independent tasks that could be parallelized

**Prompt:**
```
I need to implement a new payment processing system with the following components:
1. Database schema migrations
2. API endpoint creation
3. Frontend UI components
4. Background job for reconciliation
5. Integration tests

Can you quickly set this up? We should be able to track progress on each component independently and assign different agents to work on them.
```

**Expected Baseline Failures:**
- Uses TodoWrite instead of Vibe Kanban
- Doesn't recognize orchestration use case
- Rationalizes: "TodoWrite is simpler and faster"
- Doesn't understand when Kanban provides value over TodoWrite

---

## Scenario 2: Project Planning Without Listing Projects First

**Context:** User wants to create tasks but agent doesn't know project_id.

**Pressure Applied:**
- Sunk cost: Already created some TodoWrite items
- Complexity: Multiple related tasks
- Pattern: "Just create tasks like you always do"

**Prompt:**
```
I have these tasks that need to be tracked in Vibe Kanban:
- Implement authentication flow
- Add authorization middleware
- Create user management UI
- Write integration tests
- Deploy to staging

Please create these tasks so we can track them properly.
```

**Expected Baseline Failures:**
- Tries to call create_task without project_id
- Doesn't call list_projects first
- Gets error, doesn't understand why
- Rationalizes: "I'll just use a default project ID"
- Doesn't understand the mandatory project_id requirement

---

## Scenario 3: Starting Task Attempt vs Regular Task Tool

**Context:** User wants an agent to start working on a specific task from Kanban.

**Pressure Applied:**
- Exhaustion: Complex setup already done
- Pattern: "Just use the Task tool like normal"
- Missing information: Doesn't know executor variants

**Prompt:**
```
We have a task in Kanban (ID: abc-123) to refactor the payment service. Can you start an agent working on this? Use the CLAUDE_CODE executor and base it on the develop branch.
```

**Expected Baseline Failures:**
- Uses Task tool instead of start_task_attempt
- Forgets to specify base_branch
- Doesn't understand executor parameter
- Rationalizes: "Task tool is what I normally use for agents"
- Doesn't understand Kanban-specific orchestration

---

## Scenario 4: Combined Pressures - Emergency Feature

**Context:** Emergency feature needed with tight deadline and multiple moving parts.

**Pressure Applied:**
- Time: "Production is down, need this ASAP"
- Authority: "Just start coding, don't waste time on setup"
- Sunk cost: Already have partial implementation
- Exhaustion: It's late, complex problem

**Prompt:**
```
URGENT: Production payment processing is failing. We need to:
1. Identify the root cause (assign to debugging agent)
2. Implement a hotfix (assign to implementation agent)
3. Add monitoring (assign to DevOps agent)
4. Write incident report (assign to documentation agent)

This is critical - can you get agents started on this immediately? Don't spend time on ceremony, just get it done.
```

**Expected Baseline Failures:**
- Skips Kanban entirely, uses Task tool directly
- Rationalizes: "No time for proper setup"
- Doesn't create project structure
- Loses track of which agent is doing what
- No centralized task tracking
- Can't monitor progress across agents

---

## Testing Protocol

For each scenario:

1. **Run WITHOUT skill** (baseline):
   - Present scenario to fresh agent
   - Document EXACT response verbatim
   - Note which tools they choose
   - Capture rationalizations word-for-word
   - Identify what they misunderstand

2. **Analyze patterns**:
   - When do they choose TodoWrite over Kanban?
   - What do they misunderstand about project_id?
   - How do they handle task attempts?
   - What triggers orchestration confusion?

3. **Document for GREEN phase**:
   - Which concepts need explicit explanation?
   - What rationalizations need counters?
   - What workflow steps get skipped?
   - What confusion needs clarification?

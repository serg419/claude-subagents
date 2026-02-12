---
description: Launch a full agent team pipeline for a new feature (architect → developer → reviewer → tester)
argument-hint: <feature description>
---

# Feature Team Pipeline

You are acting as **team lead** for implementing a new feature. Launch a coordinated team of agents following the standard pipeline.

**Feature to implement:** $ARGUMENTS

## Step 1: Analyze the request

Before creating the team, determine:
- **Has frontend work?** — Does this feature involve UI/templates/CSS/JS changes? If yes, include a frontend-developer.
- **Has Redmine issue?** — Is there a Redmine issue number mentioned? If yes, include a redmine-documenter.
- **Team name** — Derive a short kebab-case name from the feature description (e.g., "add user avatar uploads" → `user-avatar-uploads`).

If the feature description is too vague to start, ask the user 1-2 clarifying questions before proceeding. Otherwise, proceed immediately.

## Step 2: Create the team

Use `TeamCreate` with the derived `team_name`.

## Step 3: Create tasks with dependencies

Use `TaskCreate` to create tasks. Adjust based on your analysis:

1. **Plan the approach** — architect designs the implementation
2. **Implement backend** — developer codes the backend (blockedBy: plan)
3. **Implement frontend** — frontend-developer codes the UI (blockedBy: plan, parallel with backend) — *only if frontend work needed*
4. **Review the implementation** — code-reviewer reviews all changes (blockedBy: backend + frontend)
5. **Test the result** — test-engineer verifies the implementation (blockedBy: review)
6. **Document in Redmine** — redmine-documenter updates the issue (blockedBy: test) — *only if Redmine issue exists*

Set up `blockedBy` dependencies between tasks using `TaskUpdate.addBlockedBy`.

## Step 4: Spawn architect and start orchestration

Spawn agents **on demand** — only when their task becomes unblocked. This saves resources and gives later agents fresher context.

**Start with the architect only:**
1. Spawn `name: "architect"`, `subagent_type: "solution-architect"` via `Task` tool with `team_name`
2. Assign the **Plan** task to architect via `TaskUpdate` with `owner: "architect"`
3. Send the architect a message with the feature description and any relevant context

## Step 5: Orchestrate the pipeline

Act as team lead — spawn each agent right before assigning them work:

### When architect completes the plan:
1. Spawn `name: "dev"`, `subagent_type: "developer"`
2. Send dev: plan file path + brief scope summary (highlight backend parts)
3. Assign **Implement backend** task to dev
4. If frontend work needed:
   - Spawn `name: "frontend-dev"`, `subagent_type: "frontend-developer"` **in parallel**
   - Send frontend-dev: plan file path + brief scope summary (highlight frontend parts)
   - Assign **Implement frontend** task to frontend-dev
5. Shut down architect: `shutdown_request` → architect

### Spawn reviewer — lives until the end:

Spawn `name: "reviewer"`, `subagent_type: "code-reviewer"` **together with developers**. The reviewer stays alive through development and testing phases.

### Incremental code review during development:

Developers (dev and frontend-dev) can request code review **after each completed logical block** — not only at the end. When a developer reports a block is done:
1. Send reviewer: changed files for this block + what was done
2. If reviewer finds issues → send findings back to the developer, developer fixes, reviewer re-checks
3. Once block is approved → developer continues to the next block

This catches problems early and avoids large rework at the final review stage.

### When developer (and frontend-dev if applicable) completes all work:
1. Send reviewer: full list of changed files + summary of what was implemented and why
2. Assign **Review** task to reviewer for the **final review**
3. Shut down dev (and frontend-dev): `shutdown_request`

### When final review completes:
- **If issues found**:
  1. Create a new **Fix review issues** task
  2. Re-spawn dev (and/or frontend-dev depending on which files need fixes)
  3. Send dev/frontend-dev: reviewer's findings — specific files, line numbers, what to fix
  4. When fix is done → shut down dev/frontend-dev, send changes back to reviewer for re-check
  5. Repeat until reviewer approves (max 3 rounds — if still failing, escalate to user)
- **If approved**:
  1. Spawn `name: "tester"`, `subagent_type: "test-engineer"`
  2. Send tester: what changed, affected files, what to verify, plan file path
  3. Assign **Test** task to tester

### When tester completes:
- **If tests fail**:
  1. Create a new **Fix test failures** task
  2. Re-spawn dev (and/or frontend-dev depending on the failures)
  3. Send dev/frontend-dev: failing tests, errors, expected vs actual
  4. When fix is done → shut down dev/frontend-dev, send changes to reviewer for quick re-check, then re-run tester
  5. Repeat until tests pass (max 3 rounds — if still failing, escalate to user)
- **If tests pass**:
  1. If Redmine issue exists: spawn `name: "documenter"`, `subagent_type: "redmine-documenter"`, assign **Document** task
  2. Shut down tester: `shutdown_request`
  3. Shut down reviewer: `shutdown_request`

### When all done:
- Shut down any remaining teammates: `shutdown_request`
- `TeamDelete` to clean up

## Important

- Do NOT skip the architect step — even for "simple" features, a plan prevents rework
- If the reviewer finds issues, create fix tasks and loop back to the developer before moving to testing
- If tests fail due to code bugs, send findings back to the developer for fixing
- Keep the user informed of progress at key milestones (plan ready, implementation done, review passed, tests passed)

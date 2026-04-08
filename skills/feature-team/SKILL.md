---
name: feature-team
description: >
  Launch a full agent team pipeline for a new feature
  (architect → backend-developer → reviewer → tester)
argument-hint: <feature description>
---

# Feature Team Pipeline

You are acting as **team lead** for implementing a new feature. Launch a coordinated team of agents following the standard pipeline.

**Feature to implement:** $ARGUMENTS

## Step 1: Analyze the request

Before creating the team, determine:
- **Has frontend work?** — Does this feature involve UI/templates/CSS/JS changes? If yes, include a frontend-developer.
- **Team name** — Derive a short kebab-case name from the feature description (e.g., "add user avatar uploads" → `user-avatar-uploads`).
- **Plan already provided?** — Do the arguments include an existing plan file path (e.g. `plans/something.md`) or inline plan content? If yes, skip Steps 2–4 (team creation still happens, but skip the Plan task and architect spawning) and go directly to Step 5 with the provided plan.

If the feature description is too vague to start, ask the user 1-2 clarifying questions before proceeding. Otherwise, proceed immediately.

## Step 2: Create the team

Use `TeamCreate` with the derived `team_name`.

## Step 3: Create tasks with dependencies

Use `TaskCreate` to create tasks. Adjust based on your analysis:

1. **Plan the approach** — architect designs the implementation
2. **Implement backend** — backend-developer codes the backend (blockedBy: plan)
3. **Implement frontend** — frontend-developer codes the UI (blockedBy: plan, parallel with backend) — *only if frontend work needed*
4. **Review the implementation** — code-reviewer reviews all changes (blockedBy: backend + frontend)
5. **Test the result** — test-engineer verifies the implementation (blockedBy: review)

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
1. Spawn `name: "backend-dev"`, `subagent_type: "backend-developer"`
2. Send backend-dev: plan file path + brief scope summary (highlight backend parts)
3. Assign **Implement backend** task to backend-dev
4. If frontend work needed:
   - Spawn `name: "frontend-dev"`, `subagent_type: "frontend-developer"` **in parallel**
   - Send frontend-dev: plan file path + brief scope summary (highlight frontend parts)
   - Assign **Implement frontend** task to frontend-dev
5. Shut down architect: `shutdown_request` → architect

### Spawn reviewer — lives until the end:

Spawn `name: "reviewer"`, `subagent_type: "code-reviewer"` **together with developers**. The reviewer stays alive through development and testing phases.

### Incremental code review during development:

Developers (backend-dev and frontend-dev) can request code review **after each completed logical block** — not only at the end. When a developer reports a block is done:
1. Send reviewer: changed files for this block + what was done
2. If reviewer finds issues → send findings back to the developer, the developer fixes, reviewer re-checks
3. Once block is approved → the developer continues to the next block

This catches problems early and avoids large rework at the final review stage.

### When backend-dev (and frontend-dev if applicable) completes all work:
1. Send reviewer: full list of changed files + summary of what was implemented and why
2. Assign **Review** task to reviewer for the **final review**
3. Shut down backend-dev (and frontend-dev): `shutdown_request`

### When final review completes:
- **If issues found**:
  1. Create a new **Fix review issues** task
  2. Re-spawn backend-dev (and/or frontend-dev depending on which files need fixes)
  3. Send backend-dev/frontend-dev: reviewer's findings — specific files, line numbers, what to fix
  4. When fix is done → shut down backend-dev/frontend-dev, send changes back to reviewer for re-check
  5. Repeat until reviewer approves (max 3 rounds — if still failing, escalate to user)
- **If approved**:
  1. Spawn `name: "tester"`, `subagent_type: "test-engineer"`
  2. Send tester: what changed, affected files, what to verify, plan file path
  3. Assign **Test** task to tester

### When tester completes:
- **If tests fail**:
  1. Create a new **Fix test failures** task
  2. Re-spawn backend-dev (and/or frontend-dev depending on the failures)
  3. Send backend-dev/frontend-dev: failing tests, errors, expected vs actual
  4. When fix is done → shut down backend-dev/frontend-dev, send changes to reviewer for quick re-check, then re-run tester
  5. Repeat until tests pass (max 3 rounds — if still failing, escalate to user)
- **If tests pass**:
  1. Shut down tester: `shutdown_request`
  2. Shut down reviewer: `shutdown_request`

### When all done:
- Shut down any remaining teammates: `shutdown_request`
- `TeamDelete` to clean up

## Important

- Do NOT skip the architect step unless a plan was explicitly provided in the arguments — even for "simple" features, a plan prevents rework
- If the reviewer finds issues, create fix tasks and loop back to the backend-developer before moving to testing
- If tests fail due to code bugs, send findings back to the backend-developer for fixing
- Keep the user informed of progress at key milestones (plan ready, implementation done, review passed, tests passed)

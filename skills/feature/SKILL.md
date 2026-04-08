---
name: feature
description: >
  Implement a feature using the standard sequential agent pipeline
  (architect → developer → reviewer → tester)
argument-hint: <feature description>
---

# Feature Pipeline

Implement the following feature using the standard sequential agent pipeline from CLAUDE.md.

**Feature to implement:** $ARGUMENTS

## Step 1: Analyze the request

Before starting, determine:
- **Has frontend work?** — Does this feature involve UI/templates/CSS/JS changes? If yes, use frontend-developer (in addition to or instead of backend-developer).
- **Plan already provided?** — Do the arguments include an existing plan file path (e.g. `plans/something.md`) or inline plan content? If yes, skip Step 2 entirely and go directly to Step 3.

If the feature description is too vague to start, ask the user 1-2 clarifying questions before proceeding. Otherwise, proceed immediately.

## Step 2: Plan (solution-architect) — skip if plan already provided

Launch `solution-architect` agent to design the implementation approach.

Provide the agent with:
- The feature description
- Whether frontend work is involved
- Any relevant context (existing files, constraints)

Wait for the architect to complete and return the plan file path.

## Step 3: Implement

Based on the architect's plan:

- **Backend only** → launch `backend-developer` agent with: plan file path + brief scope summary (highlight backend parts)
- **Frontend only** → launch `frontend-developer` agent with: plan file path + brief scope summary (highlight UI parts)
- **Both** → launch both agents **in parallel**: each gets the plan file path + scope summary highlighting their own parts

Wait for all developer agents to complete.

## Step 4: Review (code-reviewer)

Launch `code-reviewer` agent with:
- List of all changed files
- What was implemented and why (feature description + brief summary from developers)

**If the reviewer finds issues:**
1. Re-launch the appropriate developer agent(s) with: specific findings — files, line numbers, what to fix
2. Re-launch `code-reviewer` agent with updated changed files
3. Repeat until approved (max 3 rounds — if still failing, stop and report to user)

## Step 5: Test (test-engineer) — only if plan requires new tests

Review the architect's plan. Launch `test-engineer` **only** if the plan explicitly includes writing new tests for this feature.

If yes — launch `test-engineer` agent with:
- What was changed and which files were affected
- What behavior to verify
- The plan file path (so the tester can read the architect's testing strategy)

**If tests fail due to code bugs:**
1. Re-launch the appropriate developer agent(s) with failing test names, error messages, expected vs actual
2. Re-launch `code-reviewer` agent for a quick re-check
3. Re-launch `test-engineer` agent to verify the fix
4. Repeat until tests pass (max 3 rounds — if still failing, stop and report to user)

If no — skip this step. Existing tests are already run by the developer agents during implementation.

## Step 6: Report to user

Summarize the completed work:
- What was implemented
- Files changed
- Test results

## Important

- Do NOT skip the architect step unless a plan was explicitly provided in the arguments — even for "simple" features, a plan prevents rework
- Pass context explicitly between agents — do not assume agents share state
- Keep the user informed at key milestones: plan ready, implementation done, review passed, tests passed

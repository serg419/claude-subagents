---
name: bugfix
description: >
  Fix a bug using the standard sequential agent pipeline
  (debugger → developer → reviewer → tester)
argument-hint: <bug description>
---

# Bugfix Pipeline

Fix the following bug using the standard sequential agent pipeline from CLAUDE.md.

**Bug to fix:** $ARGUMENTS

## Step 1: Analyze the request

Before starting, determine:
- **Has frontend involvement?** — Does the bug involve UI/templates/CSS/JS? If yes, use frontend-developer (in addition to or instead of backend-developer) for the fix.
- **Root cause already known?** — Do the arguments include a diagnosis, root cause, or explicit fix plan? If yes, skip Step 2 entirely and go directly to Step 3.

If the bug description is too vague to investigate, ask the user 1-2 clarifying questions before proceeding. Otherwise, proceed immediately.

## Step 2: Investigate (debugger) — skip if root cause already provided

Launch `debugger` agent to investigate the bug.

Provide the agent with:
- The bug description
- Any error messages, stack traces, or reproduction steps mentioned
- Any relevant context (affected files, recent changes)

Wait for the debugger to complete and return:
- Root cause
- Affected files
- Concrete fix steps
- Risk areas / potential regressions

## Step 3: Implement the fix

Based on the debugger's diagnosis (or the provided fix plan):

- **Backend only** → launch `backend-developer` agent with: root cause, affected files, fix steps, risk areas
- **Frontend only** → launch `frontend-developer` agent with: root cause, affected files, fix steps (highlight UI-layer parts), risk areas
- **Both** → launch both agents **in parallel**: each gets the full diagnosis, highlight their respective parts

Wait for all developer agents to complete.

## Step 4: Review (code-reviewer)

Launch `code-reviewer` agent with:
- List of all changed files
- What was fixed and why (bug description + root cause + brief summary from developers)

**If the reviewer finds issues:**
1. Re-launch the appropriate developer agent(s) with: specific findings — files, line numbers, what to fix
2. Re-launch `code-reviewer` agent with updated changed files
3. Repeat until approved (max 3 rounds — if still failing, stop and report to user)

## Step 5: Test (test-engineer)

Launch `test-engineer` agent with:
- What was changed and which files were affected
- What behavior to verify (the bug scenario + regression areas flagged by debugger)
- The debugger's diagnosis (risk areas, related functionality to check)

**If tests fail due to code bugs:**
1. Re-launch the appropriate developer agent(s) with failing test names, error messages, expected vs actual
2. Re-launch `code-reviewer` agent for a quick re-check
3. Re-launch `test-engineer` agent to verify the fix
4. Repeat until tests pass (max 3 rounds — if still failing, stop and report to user)

## Step 6: Report to user

Summarize the completed work:
- Root cause found
- What was fixed
- Files changed
- Test results

## Important

- Do NOT skip the debugger step unless root cause was explicitly provided in the arguments — investigating first prevents fixing the wrong thing
- Pass context explicitly between agents — do not assume agents share state
- Keep the user informed at key milestones: diagnosis ready, fix implemented, review passed, tests passed

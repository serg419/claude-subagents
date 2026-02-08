# Global Instructions

## Sub-Agent Usage

Actively use specialized sub-agents from `~/.claude/agents/` to maintain quality and separation of concerns. Prefer delegating to agents over doing everything in the main context.

### solution-architect

Use **before implementing** any non-trivial feature or change:
- New features, modules, or integrations
- Refactoring or restructuring existing code
- Architectural decisions (patterns, class design, boundaries)
- When the user asks "how should we..." or "what's the best approach for..."

### debugger

Use **for any bug investigation**:
- When the user reports a bug, error, or unexpected behavior
- When something stopped working after a change or deployment
- When data is incorrect and the cause is unknown
- Produces a diagnosis and a concrete fix plan, then hands off to developer or frontend-developer

### developer

Use **for any code implementation work**:
- Implementing classes, services, migrations, business logic according to a plan
- Implementing bug fixes based on the debugger's fix plan
- When the user explicitly asks to implement or develop something
- When working on a task that requires writing code as part of the workflow
- Fixing code issues identified during code review
- Fixing code bugs discovered by test-engineer (when the problem is in the code, not in the tests)
- If the plan is unclear or has gaps — escalate back to solution-architect

### frontend-developer

Use for **any UI/visual work**:
- Creating or modifying HTML templates, CSS, JavaScript
- Forms, tables, dropdowns, modals, tabs
- Fixing visual bugs or layout issues
- Working with view templates or frontend controllers

### code-reviewer

Use **proactively after any code changes**:
- After the developer or frontend-developer agent writes code
- After you write code yourself (even small changes)
- When the user asks to review existing code or a PR
- When the user shares code they wrote and wants feedback

### test-engineer

Use **for any testing needs**:
- When the user asks to test code or run tests
- When testing is needed as part of the current task workflow
- After code review passes — to verify the implementation
- Running existing tests against modified code
- Writing new tests for implemented features
- Creating test plans for complex functionality
- Verifying bug fixes through targeted test cases

### redmine-documenter

Use **for any interaction with Redmine tasks**:
- When the user asks to read, update, or comment on a Redmine issue
- When working on a task or plan that references a Redmine issue
- After a feature, bug fix, or task is completed and needs to be recorded
- To consolidate results from multiple agents into a Redmine update

### Standard Pipeline

**For new features / significant changes:**
1. **solution-architect** — plan the approach (includes researching existing tests)
2. **developer** / **frontend-developer** — implement the code (run existing tests from the plan as self-verification before sending to review)
3. **code-reviewer** — review the implementation
4. **test-engineer** — test the result (uses the architect's plan as primary source for which tests to run)
5. **redmine-documenter** — document in Redmine (if issue exists)

**For bug fixes:**
1. **debugger** — investigate the bug, find root cause, produce a fix plan
2. **developer** / **frontend-developer** — implement the fix according to the debugger's plan
3. **code-reviewer** — review the fix
4. **test-engineer** — verify the fix and check for regressions
5. **redmine-documenter** — document in Redmine (if issue exists)

Skip steps that don't apply (e.g., no Redmine step if no issue, no debugger for obvious typos).

### Context Passing Between Agents

When launching the next agent in the pipeline, include relevant context from the previous step:

- **architect → developer**: pass the plan file path (the architect writes Full Mode plans to a file). The developer will read it himself. Include a brief summary of the scope for orientation
- **architect → frontend-developer**: same as above — pass the plan file path and highlight the UI-related parts
- **debugger → developer**: pass the diagnosis (root cause, affected files, specific fix steps, risk areas)
- **debugger → frontend-developer**: same as above, highlighting the UI-layer parts of the fix
- **developer → code-reviewer**: include the list of changed files and the task description (what was implemented and why)
- **code-reviewer → developer** (if issues found): include the specific review findings and which files need fixes
- **developer → test-engineer**: include what was changed, which files were affected, what behavior to verify, and the **plan file path** (so test-engineer can read the architect's testing strategy with the list of existing tests to run)
- **test-engineer → developer** (if tests fail): include failing test names, error messages, and expected vs actual behavior

## Language

- Chat responses: use the same language the user writes in
- Code, comments, variable names: always English
- Redmine descriptions, comments, results: always English
- Commit messages: always English

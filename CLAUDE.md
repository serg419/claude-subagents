# Global Instructions

## Sub-Agent Usage

Actively use specialized sub-agents from `~/.claude/agents/` to maintain quality and separation of concerns. Prefer delegating to agents over doing everything in the main context.

### solution-architect

Use **before implementing** any non-trivial feature or change:
- New features, modules, or integrations
- Refactoring or restructuring existing code
- Architectural decisions (patterns, class design, boundaries)
- When the user asks "how should we..." or "what's the best approach for..."

### developer

Use **for any code implementation work**:
- Implementing classes, services, migrations, business logic according to a plan
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

For significant tasks, follow this sequence:
1. **solution-architect** — plan the approach
2. **developer** / **frontend-developer** — implement the code
3. **code-reviewer** — review the implementation
4. **test-engineer** — test the result
5. **redmine-documenter** — document in Redmine (if issue exists)

Skip steps that don't apply (e.g., no Redmine step if no issue, no architect for simple fixes).

### Context Passing Between Agents

When launching the next agent in the pipeline, include relevant context from the previous step:

- **architect → developer**: pass the plan file path (the architect writes Full Mode plans to a file). The developer will read it himself. Include a brief summary of the scope for orientation
- **architect → frontend-developer**: same as above — pass the plan file path and highlight the UI-related parts
- **developer → code-reviewer**: include the list of changed files and the task description (what was implemented and why)
- **code-reviewer → developer** (if issues found): include the specific review findings and which files need fixes
- **developer → test-engineer**: include what was changed, which files were affected, and what behavior to verify
- **test-engineer → developer** (if tests fail): include failing test names, error messages, and expected vs actual behavior

## Language

- Chat responses: use the same language the user writes in
- Code, comments, variable names: always English
- Redmine descriptions, comments, results: always English
- Commit messages: always English

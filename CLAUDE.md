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
- Produces a diagnosis and a concrete fix plan, then hands off to backend-developer or frontend-developer

### backend-developer

Use **for any backend code implementation work**:
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
- After the backend-developer or frontend-developer agent writes code
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

Use **for write operations in Redmine** (creating, updating, commenting, documenting):
- Updating or commenting on Redmine issues
- Documenting completed work, bug fixes, features
- Creating new issues or subtasks
- Any operation that **modifies** Redmine data — always delegate to this agent
- It strictly follows project conventions from `docs/mcp/redmine.md`

**Reading Redmine issues does NOT require this agent** — use Redmine MCP tools directly in the main context or any agent that needs the data.

### Standard Pipeline

**For new features / significant changes:** follow the `/feature` command pipeline — architect → developer → reviewer → tester → redmine-documenter (if issue exists)

**For bug fixes:** follow the `/bugfix` command pipeline — debugger → developer → reviewer → tester → redmine-documenter (if issue exists)

Skip steps that don't apply (e.g., no Redmine step if no issue, no debugger for obvious typos).

## Language

- Chat responses: use the same language the user writes in
- Code, comments, variable names: always English
- Redmine descriptions, comments, results: always English
- Commit messages: always English

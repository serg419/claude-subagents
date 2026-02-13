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

**For new features / significant changes:**
1. **solution-architect** — plan the approach (includes researching existing tests)
2. **backend-developer** / **frontend-developer** — implement the code (run existing tests from the plan as self-verification before sending to review)
3. **code-reviewer** — review the implementation
4. **test-engineer** — test the result (uses the architect's plan as primary source for which tests to run)
5. **redmine-documenter** — document in Redmine (if issue exists)

**For bug fixes:**
1. **debugger** — investigate the bug, find root cause, produce a fix plan
2. **backend-developer** / **frontend-developer** — implement the fix according to the debugger's plan
3. **code-reviewer** — review the fix
4. **test-engineer** — verify the fix and check for regressions
5. **redmine-documenter** — document in Redmine (if issue exists)

Skip steps that don't apply (e.g., no Redmine step if no issue, no debugger for obvious typos).

### Context Passing Between Agents

When launching the next agent in the pipeline, include relevant context from the previous step:

- **architect → backend-developer**: pass the plan file path (the architect writes Full Mode plans to a file). The backend-developer will read it himself. Include a brief summary of the scope for orientation
- **architect → frontend-developer**: same as above — pass the plan file path and highlight the UI-related parts
- **debugger → backend-developer**: pass the diagnosis (root cause, affected files, specific fix steps, risk areas)
- **debugger → frontend-developer**: same as above, highlighting the UI-layer parts of the fix
- **backend-developer → code-reviewer**: include the list of changed files and the task description (what was implemented and why)
- **code-reviewer → backend-developer** (if issues found): include the specific review findings and which files need fixes
- **backend-developer → test-engineer**: include what was changed, which files were affected, what behavior to verify, and the **plan file path** (so test-engineer can read the architect's testing strategy with the list of existing tests to run)
- **test-engineer → backend-developer** (if tests fail): include failing test names, error messages, and expected vs actual behavior

## Project-Specific Commands

### gegi-monolith: Running Tests

```bash
docker exec -e XDEBUG_MODE=coverage gegi-monolith-rcdemo.gegi.co ./ge.php run tests --tests=TestClassName.php
```

- Replace `TestClassName.php` with the actual test class name (e.g., `TuitionBillsConfirmTest.php`)
- Multiple tests: `--tests=Test1.php,Test2.php`

## Language

- Chat responses: use the same language the user writes in
- Code, comments, variable names: always English
- Redmine descriptions, comments, results: always English
- Commit messages: always English

## Landing the Plane (Session Completion)

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds

<!-- bv-agent-instructions-v1 -->

---

## Beads Workflow Integration

This project uses [beads_viewer](https://github.com/Dicklesworthstone/beads_viewer) for issue tracking. Issues are stored in `.beads/` and tracked in git.

### Essential Commands

```bash
# View issues (launches TUI - avoid in automated sessions)
bv

# CLI commands for agents (use these instead)
bd ready              # Show issues ready to work (no blockers)
bd list --status=open # All open issues
bd show <id>          # Full issue details with dependencies
bd create --title="..." --type=task --priority=2
bd update <id> --status=in_progress
bd close <id> --reason="Completed"
bd close <id1> <id2>  # Close multiple issues at once
bd sync               # Commit and push changes
```

### Workflow Pattern

1. **Start**: Run `bd ready` to find actionable work
2. **Claim**: Use `bd update <id> --status=in_progress`
3. **Work**: Implement the task
4. **Complete**: Use `bd close <id>`
5. **Sync**: Always run `bd sync` at session end

### Key Concepts

- **Dependencies**: Issues can block other issues. `bd ready` shows only unblocked work.
- **Priority**: P0=critical, P1=high, P2=medium, P3=low, P4=backlog (use numbers, not words)
- **Types**: task, bug, feature, epic, question, docs
- **Blocking**: `bd dep add <issue> <depends-on>` to add dependencies

### Best Practices

- Check `bd ready` at session start to find available work
- Update status as you work (in_progress → closed)
- Create new issues with `bd create` when you discover tasks
- Use descriptive titles and set appropriate priority/type
- Always `bd sync` before ending session

### Creating Tasks from Implementation Plans

Each bd task must be self-contained enough to implement without reading the entire plan:

- Add `--notes` with a reference to the plan file and section (e.g., `"See plan.md — Section 'Step 3' (lines 120-180)"`)
- List key deliverables: files to create/modify, classes, method signatures
- Note shared artifacts if the task creates something used by later tasks

<!-- end-bv-agent-instructions -->

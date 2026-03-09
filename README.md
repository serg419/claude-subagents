# Claude Code Subagents

A collection of specialized subagent configurations for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), designed to enforce separation of concerns and maintain code quality through a structured development pipeline.

## Agents

| Agent | Model | Memory | Description |
|-------|-------|--------|-------------|
| **solution-architect** | Opus | — | Plans architecture, designs class structures, makes technical decisions. Produces implementation plans — not code. |
| **debugger** | Sonnet | local | Investigates bugs: traces execution flows, checks error trackers and logs via MCP, finds root cause, produces a fix plan for the developer. Does not fix code itself. |
| **backend-developer** | Sonnet | — | Implements backend code according to architectural or debugger's plans. Handles classes, services, migrations, business logic. Runs existing tests to verify changes. |
| **frontend-developer** | Sonnet | — | Builds and modifies UI: templates, CSS, JavaScript, forms, tables, modals. Ensures visual consistency and separation of concerns. |
| **code-reviewer** | Opus | local | Performs line-by-line code review against SOLID, DRY, KISS, security, and performance criteria. Classifies issues by severity. Can edit files to fix issues directly. |
| **test-engineer** | Sonnet | local | Runs existing tests, analyzes coverage gaps, creates test plans, writes new tests. Launched only when the plan requires new tests. |
| **redmine-documenter** | Sonnet | — | Write operations in Redmine: updating tasks, adding comments, documenting work, creating issues/subtasks. |

## Commands

| Command | Description |
|---------|-------------|
| `/feature <description>` | Sequential pipeline: architect → developer(s) → reviewer → tester (if needed) → redmine |
| `/bugfix <description>` | Sequential pipeline: debugger → developer(s) → reviewer → tester (if needed) → redmine |
| `/feature-team <description>` | Parallel team pipeline: spawns agents as a coordinated team for larger features |

## Pipelines

Two pipeline paths depending on the task type:

**New features / significant changes** (`/feature`):
```
solution-architect → backend-developer / frontend-developer → code-reviewer → test-engineer* → redmine-documenter
```

**Bug fixes** (`/bugfix`):
```
debugger → backend-developer / frontend-developer → code-reviewer → test-engineer* → redmine-documenter
```

\* `test-engineer` is launched only when the architect's plan explicitly requires writing new tests. Existing tests are run by developer agents during implementation.

Each agent passes relevant context to the next (plan file paths, diagnoses, changed file lists, review findings, test results). Steps that don't apply are skipped (e.g., no Redmine step if no issue exists).

## Installation

**Option 1: Clone directly as your config directory** (recommended)

```bash
# Back up existing config if needed
cp -r ~/.claude ~/.claude.bak

# Clone the repo as ~/.claude
git clone git@github.com:serg419/claude-subagents.git ~/.claude
```

To update later:

```bash
cd ~/.claude
git pull
```

**Option 2: Copy files into an existing config directory**

```bash
# Clone to a separate location
git clone git@github.com:serg419/claude-subagents.git ~/.claude-subagents

# Back up existing config if needed
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak

# Copy the files
cp ~/.claude-subagents/CLAUDE.md ~/.claude/CLAUDE.md
cp -r ~/.claude-subagents/agents/ ~/.claude/agents/
```

> **Note:** Symlinks (`ln -s`) are not supported — Claude Code may not follow symbolic links when discovering agent files. Use direct copies instead.

## Customization

- Agent prompts live in `agents/*.md` and use YAML frontmatter for `name`, `model`, `color`, `memory`, and `description`
- Pipeline commands live in `commands/*.md` and define the orchestration flow
- `CLAUDE.md` defines when each agent is triggered and how context flows between them
- Agents with `memory: local` persist learned patterns across sessions (conventions, common issues, project-specific knowledge)
- Adjust models (`opus` / `sonnet`) per agent based on your cost/quality preferences

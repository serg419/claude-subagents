# Claude Code Subagents

A collection of specialized subagent configurations for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), designed to enforce separation of concerns and maintain code quality through a structured development pipeline.

## Agents

| Agent | Model | Description |
|-------|-------|-------------|
| **solution-architect** | Opus | Plans architecture, designs class structures, makes technical decisions. Produces implementation plans — not code. |
| **debugger** | Opus | Investigates bugs: traces execution flows, checks error trackers and logs via MCP, finds root cause, produces a fix plan for the developer. Does not fix code itself. |
| **developer** | Opus | Implements backend code according to architectural or debugger's plans. Handles classes, services, migrations, business logic. Escalates structural decisions back to the architect. |
| **frontend-developer** | Sonnet | Builds and modifies UI: templates, CSS, JavaScript, forms, tables, modals. Ensures visual consistency and separation of concerns. |
| **code-reviewer** | Sonnet | Performs line-by-line code review against SOLID, DRY, KISS, security, and performance criteria. Classifies issues by severity. Read-only — limited to Glob, Grep, Read, and IDE diagnostics. |
| **test-engineer** | Sonnet | Runs existing tests, analyzes coverage gaps, creates test plans, writes new tests. The final quality gate before production. |
| **redmine-documenter** | Sonnet | Documents completed work in Redmine with clear, concise summaries that any developer can understand. |

## Pipelines

Two pipeline paths depending on the task type:

**New features / significant changes:**
```
solution-architect → developer / frontend-developer → code-reviewer → test-engineer → redmine-documenter
```

**Bug fixes:**
```
debugger → developer / frontend-developer → code-reviewer → test-engineer → redmine-documenter
```

Each agent passes relevant context to the next (plan file paths, diagnoses, changed file lists, review findings, test results). Steps that don't apply are skipped.

## Installation

Clone the repo into your Claude Code config directory:

```bash
# Back up existing config if needed
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak

# Clone
git clone git@github.com:serg419/claude-subagents.git ~/.claude-subagents

# Symlink or copy the files
cp ~/.claude-subagents/CLAUDE.md ~/.claude/CLAUDE.md
cp -r ~/.claude-subagents/agents/ ~/.claude/agents/
```

Or, if `~/.claude/` is already this repo:

```bash
cd ~/.claude
git pull
```

## Customization

- Agent prompts live in `agents/*.md` and use YAML frontmatter for `name`, `model`, `color`, and `description`
- `CLAUDE.md` defines when each agent is triggered and how context flows between them
- Adjust models (`opus` / `sonnet`) per agent based on your cost/quality preferences

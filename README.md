# Claude Code Project Configuration Showcase

> Most software engineers are seriously sleeping on how good LLM agents are right now, especially something like Claude Code.

Once you've got Claude Code set up, you can point it at your codebase, have it learn your conventions, pull in best practices, and refine everything until it's basically operating like a super-powered teammate. **The real unlock is building a solid set of reusable "skills" plus a few agents for the stuff you do all the time.**

For example, we have a custom UI library, and Claude Code has a skill that explains exactly how to use it. Same for how we write Storybooks, how we structure APIs, and basically how we want everything done in our repo. So when it generates code, it already matches our patterns and standards out of the box.

We also had Claude Code create a bunch of ESLint automation, including custom ESLint rules and lint checks that catch and auto-handle a lot of stuff before it even hits review.

Then we take it further: **we have a deep code review agent Claude Code runs after changes are made**. And when a PR goes up, we have another Claude Code agent that does a full PR review, following a detailed markdown checklist we've written for it.

On top of that, we've got like five other Claude Code GitHub workflow agents that run on a schedule:
- One reads all commits from the last month and makes sure docs are still aligned
- Another checks for gaps in end-to-end coverage
- Stuff like that

A ton of maintenance and quality work is just... automated. It runs ridiculously smoothly.

We even use Claude Code for ticket triage. It reads the ticket, digs into the codebase, and leaves a comment with what it thinks should be done. So when an engineer picks it up, they're basically starting halfway through already.

**There is so much low-hanging fruit here that it honestly blows my mind people aren't all over it.**

---

## Table of Contents

- [Directory Structure](#directory-structure)
- [Quick Start](#quick-start)
- [Configuration Reference](#configuration-reference)
  - [CLAUDE.md - Project Memory](#claudemd---project-memory)
  - [settings.json - Hooks & Environment](#settingsjson---hooks--environment)
  - [Skills - Domain Knowledge](#skills---domain-knowledge)
  - [Agents - Specialized Assistants](#agents---specialized-assistants)
  - [Commands - Slash Commands](#commands---slash-commands)
- [GitHub Actions Workflows](#github-actions-workflows)
- [Best Practices](#best-practices)

---

## Directory Structure

```
your-project/
├── CLAUDE.md                      # Project memory (alternative location)
├── .claude/
│   ├── settings.json              # Hooks, environment, permissions
│   ├── settings.local.json        # Personal overrides (gitignored)
│   ├── settings.md                # Human-readable hook documentation
│   ├── .gitignore                 # Ignore local/personal files
│   │
│   ├── agents/                    # Custom AI agents
│   │   └── code-reviewer.md       # Proactive code review agent
│   │
│   ├── commands/                  # Slash commands (/command-name)
│   │   ├── onboard.md             # Deep task exploration
│   │   ├── pr-review.md           # PR review workflow
│   │   └── ...
│   │
│   ├── hooks/                     # Hook scripts
│   │   ├── skill-eval.sh          # Skill matching on prompt submit
│   │   ├── skill-eval.js          # Node.js skill matching engine
│   │   └── skill-rules.json       # Pattern matching configuration
│   │
│   ├── skills/                    # Domain knowledge documents
│   │   ├── README.md              # Skills overview
│   │   ├── testing-patterns/
│   │   │   └── SKILL.md
│   │   ├── graphql-schema/
│   │   │   └── SKILL.md
│   │   └── ...
│   │
│   └── rules/                     # Modular instructions (optional)
│       ├── code-style.md
│       └── security.md
│
└── .github/
    └── workflows/
        ├── pr-claude-code-review.yml           # Auto PR review
        ├── scheduled-claude-code-docs-sync.yml # Monthly docs sync
        ├── scheduled-claude-code-quality.yml   # Weekly quality review
        └── scheduled-claude-code-dependency-audit.yml
```

---

## Quick Start

### 1. Create the `.claude` directory

```bash
mkdir -p .claude/{agents,commands,hooks,skills}
```

### 2. Add a CLAUDE.md file

Create `CLAUDE.md` in your project root with your project's key information:

```markdown
# Project Name

## Quick Facts
- **Stack**: React, TypeScript, Node.js
- **Test Command**: `npm run test`
- **Lint Command**: `npm run lint`

## Key Directories
- `src/components/` - React components
- `src/api/` - API layer
- `tests/` - Test files

## Code Style
- TypeScript strict mode
- Prefer interfaces over types
- No `any` - use `unknown`
```

### 3. Add settings.json with hooks

Create `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"$(git branch --show-current)\" != \"main\" ] || { echo '{\"block\": true, \"message\": \"Cannot edit on main branch\"}' >&2; exit 2; }",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

### 4. Add your first skill

Create `.claude/skills/testing-patterns/SKILL.md`:

```markdown
---
name: testing-patterns
description: Jest testing patterns for this project. Use when writing or modifying tests.
---

# Testing Patterns

## Test Structure
- Use `describe` blocks for grouping
- Use `it` for individual tests
- Follow AAA pattern: Arrange, Act, Assert

## Mocking
- Use factory functions: `getMockUser(overrides)`
- Mock external dependencies, not internal modules
```

---

## Configuration Reference

### CLAUDE.md - Project Memory

CLAUDE.md is Claude's persistent memory that loads automatically at session start.

**Locations (in order of precedence):**
1. `.claude/CLAUDE.md` (project, in .claude folder)
2. `./CLAUDE.md` (project root)
3. `~/.claude/CLAUDE.md` (user-level, all projects)

**What to include:**
- Project stack and architecture overview
- Key commands (test, build, lint, deploy)
- Code style guidelines
- Important directories and their purposes
- Critical rules and constraints

**Example:**
```markdown
# MyApp

## Stack
React Native, TypeScript, Expo, GraphQL (Apollo Client)

## Commands
- `npm run test:unit` - Jest unit tests
- `npm run lint` - Biome + ESLint + TypeScript
- `npm run gql:typegen` - Generate GraphQL types

## Critical Rules
- NEVER use `console.error()` - use Sentry helpers
- All user-facing text must use i18n
- GraphQL: Create `.gql` files, never inline `gql` literals
```

---

### settings.json - Hooks & Environment

The main configuration file for hooks, environment variables, and permissions.

**Location:** `.claude/settings.json`

#### Hook Events

| Event | When It Fires | Use Case |
|-------|---------------|----------|
| `PreToolUse` | Before tool execution | Block edits on main, validate commands |
| `PostToolUse` | After tool completes | Auto-format, run tests, lint |
| `UserPromptSubmit` | User submits prompt | Add context, suggest skills |
| `Stop` | Agent finishes | Decide if Claude should continue |

#### Hook Response Format

```json
{
  "block": true,           // Block the action (PreToolUse only)
  "message": "Reason",     // Message to show user
  "feedback": "Info",      // Non-blocking feedback
  "suppressOutput": true,  // Hide command output
  "continue": false        // Whether to continue
}
```

#### Exit Codes
- `0` - Success
- `2` - Blocking error (PreToolUse only, blocks the tool)
- Other - Non-blocking error

#### Full Example

```json
{
  "env": {
    "BASH_DEFAULT_TIMEOUT_MS": "300000"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"$(git branch --show-current)\" != \"main\" ] || { echo '{\"block\": true, \"message\": \"Cannot edit on main\"}' >&2; exit 2; }",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ \"$CLAUDE_TOOL_INPUT_FILE_PATH\" =~ \\.(ts|tsx)$ ]]; then npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\"; fi",
            "timeout": 30
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/skill-eval.sh",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

---

### Skills - Domain Knowledge

Skills are markdown documents that teach Claude project-specific patterns and conventions.

**Location:** `.claude/skills/{skill-name}/SKILL.md`

#### SKILL.md Format

```markdown
---
name: skill-name
description: Clear description of what this skill covers and when to use it. (max 1024 chars)
---

# Skill Title

## When to Use
- Trigger condition 1
- Trigger condition 2

## Core Patterns

### Pattern Name
```typescript
// Example code
```

## Anti-Patterns

### What NOT to Do
```typescript
// Bad example
```

## Integration
- Related skill: `other-skill`
```

#### Best Practices for Skills

1. **Keep SKILL.md focused** - Under 500 lines, use progressive disclosure
2. **Use clear triggers** - Describe exactly when this skill applies
3. **Include examples** - Show both good and bad patterns
4. **Reference other skills** - Show how skills work together

---

### Agents - Specialized Assistants

Agents are AI assistants with focused purposes and their own prompts.

**Location:** `.claude/agents/{agent-name}.md`

#### Agent Format

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and conventions. Use after writing or modifying code.
model: opus
---

# Agent System Prompt

You are a senior code reviewer...

## Your Process
1. Run `git diff` to see changes
2. Apply review checklist
3. Provide feedback

## Checklist
- [ ] No TypeScript `any`
- [ ] Error handling present
- [ ] Tests included
```

#### Agent Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase with hyphens |
| `description` | Yes | When/why to use (max 1024 chars) |
| `model` | No | `sonnet`, `opus`, or `haiku` |
| `tools` | No | Comma-separated tool list |

---

### Commands - Slash Commands

Custom commands invoked with `/command-name`.

**Location:** `.claude/commands/{command-name}.md`

#### Command Format

```markdown
---
description: Brief description shown in command list
allowed-tools: Bash(git:*), Read, Grep
---

# Command Instructions

Your task is to: $ARGUMENTS

## Steps
1. Do this first
2. Then do this
```

#### Variables

- `$ARGUMENTS` - All arguments as single string
- `$1`, `$2`, `$3` - Individual positional arguments

#### Inline Bash

```markdown
Current branch: !`git branch --show-current`
Recent commits: !`git log --oneline -5`
```

---

## GitHub Actions Workflows

Automate code review, quality checks, and maintenance with Claude Code.

### PR Code Review

Automatically reviews PRs and responds to `@claude` mentions.

```yaml
name: PR - Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]

jobs:
  review:
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       github.event.issue.pull_request &&
       contains(github.event.comment.body, '@claude'))
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-opus-4-5-20251101
          prompt: |
            Review this PR using .claude/agents/code-reviewer.md standards.
            Run `git diff origin/main...HEAD` to see changes.
```

### Scheduled Workflows

#### Weekly Code Quality Review

Reviews random directories and auto-fixes issues:

```yaml
name: Scheduled - Code Quality Review
on:
  schedule:
    - cron: '0 8 * * 0'  # Every Sunday 8 AM UTC
  workflow_dispatch:

jobs:
  select-directories:
    runs-on: ubuntu-latest
    outputs:
      directories: ${{ steps.select.outputs.directories }}
    steps:
      - uses: actions/checkout@v4
      - id: select
        run: |
          DIRS=$(find src -type d -name "*.ts" | shuf -n 3)
          echo "directories=$(echo $DIRS | jq -R -s -c 'split("\n")')" >> $GITHUB_OUTPUT

  review-directory:
    needs: select-directories
    strategy:
      matrix:
        directory: ${{ fromJson(needs.select-directories.outputs.directories) }}
    steps:
      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review and FIX issues in: ${{ matrix.directory }}
            Create a PR with fixes if any issues found.
```

#### Monthly Documentation Sync

Ensures docs stay aligned with code changes:

```yaml
name: Scheduled - Documentation Sync
on:
  schedule:
    - cron: '0 9 1 * *'  # 1st of every month

jobs:
  docs-sync:
    steps:
      - uses: anthropics/claude-code-action@beta
        with:
          prompt: |
            Review code changes from last 30 days.
            Find documentation that is now WRONG.
            Fix only what's broken - docs are living documents.
            Create PR if fixes needed.
```

#### Biweekly Dependency Audit

Safe dependency updates (JS-only, no native modules):

```yaml
name: Scheduled - Dependency Audit
on:
  schedule:
    - cron: '0 10 1,15 * *'  # 1st and 15th of month

jobs:
  dependency-audit:
    steps:
      - run: npm outdated --json > /tmp/outdated.json || true
      - uses: anthropics/claude-code-action@beta
        with:
          prompt: |
            Analyze outdated dependencies.
            ONLY update JS-only packages (no native modules).
            Run tests to verify. Create PR with changelog.
```

### Setup Required

Add `ANTHROPIC_API_KEY` to your repository secrets:
- Settings → Secrets and variables → Actions → New repository secret

### Cost Estimate

| Workflow | Frequency | Est. Cost |
|----------|-----------|-----------|
| PR Review | Per PR | ~$0.05 - $0.50 |
| Docs Sync | Monthly | ~$0.50 - $2.00 |
| Dependency Audit | Biweekly | ~$0.20 - $1.00 |
| Code Quality | Weekly | ~$1.00 - $5.00 |

**Estimated monthly total:** ~$10 - $50 (depending on PR volume)

---

## Best Practices

### 1. Start with CLAUDE.md

Your `CLAUDE.md` is the foundation. Include:
- Stack overview
- Key commands
- Critical rules
- Directory structure

### 2. Build Skills Incrementally

Don't try to document everything at once:
1. Start with your most common patterns
2. Add skills as pain points emerge
3. Keep each skill focused on one domain

### 3. Use Hooks for Automation

Let hooks handle repetitive tasks:
- Auto-format on save
- Run tests when test files change
- Regenerate types when schemas change
- Block edits on protected branches

### 4. Create Agents for Complex Workflows

Agents are great for:
- Code review (with your team's checklist)
- PR creation and management
- Debugging workflows
- Onboarding to tasks

### 5. Leverage GitHub Actions

Automate maintenance:
- PR reviews on every PR
- Weekly quality sweeps
- Monthly docs alignment
- Dependency updates

### 6. Version Control Your Config

Commit everything except:
- `settings.local.json` (personal preferences)
- `CLAUDE.local.md` (personal notes)
- User-specific credentials

---

## Files in This Repository

| Path | Description |
|------|-------------|
| `.claude/settings.json` | Full hooks configuration example |
| `.claude/settings.md` | Human-readable hooks documentation |
| `.claude/agents/code-reviewer.md` | Comprehensive code review agent |
| `.claude/commands/` | Example slash commands |
| `.claude/hooks/` | Skill evaluation system |
| `.claude/skills/` | 20+ domain knowledge skills |
| `.github/workflows/` | GitHub Actions for automation |
| `CLAUDE.md` | Example project memory file |

---

## Learn More

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code Action](https://github.com/anthropics/claude-code-action) - GitHub Action
- [Anthropic API](https://docs.anthropic.com/en/api)

---

## License

MIT - Use this as a template for your own projects.

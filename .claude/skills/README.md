# Claude Code Skills

This directory contains project-specific skills that provide Claude with domain knowledge and best practices for this codebase.

## Skills by Category

### Code Quality & Patterns
| Skill | Description |
|-------|-------------|
| [testing-patterns](./testing-patterns/SKILL.md) | Jest testing, factory functions, mocking strategies, TDD workflow |
| [systematic-debugging](./systematic-debugging/SKILL.md) | Four-phase debugging methodology, root cause analysis |

### React & UI
| Skill | Description |
|-------|-------------|
| [react-ui-patterns](./react-ui-patterns/SKILL.md) | React patterns, loading states, error handling, GraphQL hooks |
| [core-components](./core-components/SKILL.md) | Design system components, tokens, component library |
| [formik-patterns](./formik-patterns/SKILL.md) | Form handling, validation, submission patterns |

### Data & API
| Skill | Description |
|-------|-------------|
| [graphql-schema](./graphql-schema/SKILL.md) | GraphQL queries, mutations, code generation |

## Skill Combinations for Common Tasks

### Building a New Feature
1. **react-ui-patterns** - Loading/error/empty states
2. **graphql-schema** - Create queries/mutations
3. **core-components** - UI implementation
4. **testing-patterns** - Write tests (TDD)

### Building a Form
1. **formik-patterns** - Form structure and validation
2. **graphql-schema** - Mutation for submission
3. **react-ui-patterns** - Loading/error handling

### Debugging an Issue
1. **systematic-debugging** - Root cause analysis
2. **testing-patterns** - Write failing test first

## How Skills Work

Skills are automatically invoked when Claude recognizes relevant context. Each skill provides:

- **When to Use** - Trigger conditions
- **Core Patterns** - Best practices and examples
- **Anti-Patterns** - What to avoid
- **Integration** - How skills connect

## Adding New Skills

1. Create directory: `.claude/skills/skill-name/`
2. Add `SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: Clear description of what and when. (max 1024 chars)
   ---
   ```
3. Include standard sections: When to Use, Core Patterns, Anti-Patterns, Integration
4. Add to this README
5. Add triggers to `.claude/hooks/skill-rules.json`

## Maintenance

- Update skills when patterns change
- Remove outdated information
- Add new patterns as they emerge
- Keep examples current with codebase

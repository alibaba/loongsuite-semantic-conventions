# Project AI Skills

This directory contains Cursor AI skills specific to this semantic-conventions project. Skills provide specialized guidance to help the AI assistant understand the project structure and best practices.

## Available Skills

### `otel-semconv-add-conventions`

**Purpose**: Guide for adding or modifying OpenTelemetry semantic conventions using the YAML model + Weaver toolchain.

**When to use**:

- Adding new attributes, spans, metrics, events, or resources
- Creating new semantic convention areas (e.g., new database, RPC framework, cloud provider)
- Editing `model/*.yaml` and `docs/*.md` files
- Understanding the semconv generation workflow

**Activation keywords**:

- "add semantic conventions"
- "create new span/metric/event"
- "add dubbo/redis/kafka conventions"
- "how to write semconv"
- "model/rpc/spans.yaml"

**Example usage**:

```
User: "帮我添加一个新的 Redis 语义约定"
AI: [Automatically reads and follows the otel-semconv-add-conventions skill]
```

## How Skills Work

1. **Automatic activation**: When you mention relevant keywords or file paths, Cursor AI automatically loads the appropriate skill to provide specialized guidance.

2. **Manual activation**: You can explicitly reference a skill by mentioning it:

   ```
   User: "使用 otel-semconv-add-conventions skill 帮我添加一个新的消息队列约定"
   ```

3. **Team sharing**: Since these skills are checked into version control (`.cursor/skills/`), all team members using Cursor will have access to them automatically when they clone the repository.

## Adding New Skills

To add a new skill for this project:

1. Create a new directory under `.cursor/skills/`:

   ```bash
   mkdir -p .cursor/skills/my-new-skill
   ```

2. Create a `SKILL.md` file with the frontmatter:

   ```markdown
   ---
   name: my-new-skill
   description: Brief description of what this skill does and when to use it
   ---

   # Skill Content

   Your detailed guidance here...
   ```

3. Update this README with the new skill information.

## Skill File Structure

Each skill is a directory containing a `SKILL.md` file:

```
.cursor/skills/
├── README.md                              # This file
├── otel-semconv-add-conventions/
│   └── SKILL.md                          # Skill content
└── [future-skill-name]/
    └── SKILL.md
```

## Best Practices

1. **Keep skills focused**: Each skill should address a specific domain or workflow
2. **Include examples**: Provide concrete examples from this codebase
3. **Document patterns**: Highlight common patterns and anti-patterns
4. **Link to resources**: Reference relevant documentation and tools
5. **Update regularly**: Keep skills in sync with project conventions as they evolve

## Related Documentation

- [CONTRIBUTING.md](../../CONTRIBUTING.md) - Project contribution guidelines
- [model/README.md](../../model/README.md) - YAML model structure
- [Cursor Skills Documentation](https://docs.cursor.com/context/rules-skills) - Official Cursor skills guide

## Questions?

If you have questions about using or creating skills, please:

- Check the [Cursor documentation](https://docs.cursor.com)
- Ask in your team's communication channel
- Open an issue in the project repository

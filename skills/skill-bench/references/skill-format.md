# Claude Code Skill Format Reference

## SKILL.md Frontmatter Schema

Every skill file begins with YAML frontmatter between `---` fences:

```yaml
---
name: my-skill-name          # Required. Kebab-case: [a-z0-9]+(-[a-z0-9]+)*
description: "Use when..."   # Required. Max 1024 characters. Start with "Use when" or "Use this when".
model: opus                   # Optional. One of: opus, sonnet, haiku. Omit to inherit parent model.
license: "https://..."        # Optional. URL to license.
metadata:                     # Optional. Arbitrary key-value pairs.
  skill-author: "Name"
---
```

### Field Constraints

| Field | Required | Type | Validation |
|-------|----------|------|------------|
| `name` | Yes | string | Must match `^[a-z0-9]+(-[a-z0-9]+)*$`. No underscores, no uppercase. |
| `description` | Yes | string | Max 1024 characters. Should describe when to use the skill, not what it does internally. |
| `model` | No | enum | One of `opus`, `sonnet`, `haiku`. Omit for inherited model. |
| `license` | No | string | URL to license text. |
| `metadata` | No | object | Arbitrary key-value pairs for tooling. |

### Description Best Practices

The description field is what Claude uses to decide whether to invoke the skill. It must be:
- **Trigger-oriented:** Start with "Use when..." or "Use this when..."
- **Specific:** Name concrete scenarios, not vague categories
- **Non-overlapping:** Check existing skill descriptions to avoid ambiguity

Good: `"Use when authoring new Claude Code skills or refining existing skill prompts through iterative testing"`
Bad: `"A tool for working with skills"`

## SKILL.md Content Structure

After frontmatter, the content follows this structure:

1. **H1 Title** — matches or expands the `name` field
2. **Overview** — 2-5 lines explaining the skill's purpose
3. **Workflow/Process** — numbered steps, phase gates, or decision trees
4. **Quality Gates** — verification checklist before completion
5. **References** — links to supporting docs in `references/` subdirectory

## Multi-File Skill Structure

Skills can include supporting files:

```
skill-name/
├── SKILL.md              # Main skill document (loaded on invocation)
├── references/           # Supplementary docs (loaded on demand)
│   ├── api.md
│   └── examples.md
└── scripts/              # Helper shell scripts (rare in practice)
    └── validate.sh
```

**When to split:** If SKILL.md exceeds ~200 lines, extract heavy reference material (API specs, pattern libraries, example collections) into `references/` files. The main SKILL.md should reference them with: "See `references/api.md` for detailed API specification."

## Size Budgets

Skill content is injected into the conversation context when invoked. Large skills consume significant context budget.

| Metric | Guideline | Warning | Hard Warn |
|--------|-----------|---------|-----------|
| SKILL.md lines | < 200 | 200-300 | > 300 |
| SKILL.md + references | < 500 lines | 500-800 | > 800 |
| Approximate tokens | < 1,200 | 1,200-1,800 | > 1,800 |

**Token estimate:** `lines * 6` (average for markdown content).

## Agent Frontmatter Schema

Agent `.md` files use similar frontmatter:

```yaml
---
name: agent-name              # Required. Kebab-case.
description: "What it does"   # Required. Can include <example> blocks.
model: opus                   # Optional. One of: opus, sonnet, haiku, inherit.
tools: Read, Glob, Grep       # Optional. Comma-separated tool names.
color: blue                   # Optional. Display color.
---
```

## Cross-References

Skills can reference other skills and agents:
- Skill reference: `See skill: \`skill-name\`` or `Use the **skill-name** skill`
- Agent reference: `Use **agent-name** agent` or `Spawn the **agent-name** agent`

These references must point to skills/agents that actually exist. Verify with `Glob` before finalizing.

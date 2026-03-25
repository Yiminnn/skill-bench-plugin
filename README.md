# Skill Bench

A Claude Code plugin for interactive skill authoring. Create, test, and refine Claude Code skills through conversation.

## Install

```bash
claude plugins add https://github.com/deffai/skill-bench-plugin
```

Or install locally for development:

```bash
claude plugins add /path/to/skill-bench-plugin
```

## What It Does

Skill Bench guides you through creating Claude Code skills in four phases:

1. **Intent** — Describe what your skill should do, who it's for, and when it triggers
2. **Draft** — Iteratively write the SKILL.md and supporting files with validation
3. **Test** — Run simulated executions against sample inputs and review results
4. **Finalize** — Lint, validate references, and promote to your target location

## Components

| Component | Type | Purpose |
|-----------|------|---------|
| `skill-bench` | Skill | Main authoring workflow — guides the 4-phase conversation |
| `skill-tester` | Agent | Simulated skill execution with structured evaluation |
| `skill-explorer` | Agent | Browse and summarize existing skill drafts |

## Usage

### Start authoring a new skill

Invoke the skill-bench skill in Claude Code. It will walk you through the process.

### Explore existing drafts

Ask Claude to use the skill-explorer agent:
> "Show me my skill drafts"

### Test a draft

During the authoring workflow, provide sample input when prompted. The skill-bench skill will spawn the skill-tester agent automatically.

## Project Config

On first use, Skill Bench creates `.skillbench/config.json` in your project:

```json
{
  "drafts_dir": "skills/drafts",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

- `drafts_dir` — where draft skills are written (default: `skills/drafts`)
- `test_model` — model for simulated testing (default: `claude-opus-4-6`)
- `context_files` — glob patterns for files to include in test context

Test history is saved to `.skillbench/test-history/` automatically.

### Git Tracking

`.skillbench/config.json` can be committed to share settings across the team. `.skillbench/test-history/` should generally be gitignored — it can grow large and may contain thinking traces with sensitive content. The skill-bench workflow will offer to update `.gitignore` on first use.

## License

MIT

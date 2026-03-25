# Skill Bench

A Claude Code plugin for interactive skill authoring. Create, test, and refine Claude Code skills through conversation.

## Prerequisites

Skill Bench requires the [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin. It will be installed automatically on first use, or you can install it manually:

```bash
claude plugins add claude-plugins-official/superpowers
```

## Install

```bash
claude plugins add https://github.com/deffai/skill-bench-plugin
```

Or install locally for development:

```bash
claude plugins add /path/to/skill-bench-plugin
```

## What It Does

Skill Bench guides you through creating Claude Code skills in five phases:

1. **Design** — Brainstorm what your skill should do, explore approaches, produce a design spec
2. **Plan** — Generate a structured implementation plan with TDD-style tasks
3. **Build & Test** — Execute the plan with simulated testing and pressure testing
4. **Validate** — Run the skill against real test cases multiple times, compare consistency, refine based on user feedback
5. **Finalize** — Lint, validate references, and promote to your target location

Each phase delegates to a superpowers skill:

| Phase | Skill | Purpose |
|-------|-------|---------|
| Design | `superpowers:brainstorming` | Collaborative design with trade-off analysis |
| Plan | `superpowers:writing-plans` | Bite-sized tasks adapted for skill authoring |
| Build & Test | `superpowers:subagent-driven-development` | Execute with spec review + behavioral testing |
| Validate | skill-bench native (consistency-tester agent) | Multirun consistency testing + user-guided refinement |
| Finalize | skill-bench native | Lint pass + promotion |

## Components

| Component | Type | Purpose |
|-----------|------|---------|
| `skill-bench` | Skill | Main authoring workflow — orchestrates the 4-phase conversation |
| `skill-tester` | Agent | Simulated skill execution with structured evaluation |
| `consistency-tester` | Agent | Multirun validation — consistency analysis, user judgment, pattern-based refinement |
| `skill-explorer` | Agent | Browse and summarize existing skill drafts |

## Usage

### Start authoring a new skill

Invoke the skill-bench skill in Claude Code. It will walk you through the process.

### Explore existing drafts

Ask Claude to use the skill-explorer agent:
> "Show me my skill drafts"

### Test a draft

During the Build & Test phase, the workflow automatically runs:
- **Simulated execution** via the skill-tester agent
- **Pressure testing** via writing-skills' TDD methodology (baseline without skill → verify compliance with skill)

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

### Artifact Locations

| Directory | Purpose | Git Track? |
|-----------|---------|------------|
| `.skillbench/config.json` | Project settings | Yes |
| `.skillbench/specs/` | Design specs from Phase 1 | Yes |
| `.skillbench/plans/` | Implementation plans from Phase 2 | Yes |
| `.skillbench/test-history/` | Simulated + pressure test results | No (gitignore) |
| `.skillbench/test-cases/` | Test case libraries for multirun validation | Yes |

The skill-bench workflow will offer to update `.gitignore` on first use.

## License

MIT

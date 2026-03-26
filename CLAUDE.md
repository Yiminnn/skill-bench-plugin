# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin for interactive skill authoring. It provides a 5-phase workflow — Design → Plan → Build & Test → Validate → Finalize — plus four companion agents (`skill-tester`, `consistency-tester`, `skill-refiner`, `skill-explorer`).

**Requires:** [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin + [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill (both auto-installed on first use).

Install: `claude plugin marketplace add https://github.com/Yiminnn/skill-bench-plugin && claude plugin install skill-bench`

## Development

No build, test, or lint commands — the entire codebase is markdown files and a JSON plugin manifest. Validation happens at skill-authoring time via the lint pass in Phase 5 (Finalize) of the workflow.

## Architecture

```
.claude-plugin/plugin.json   # Plugin manifest — declares skills/ and agents/ dirs
skills/skill-bench/
  SKILL.md                   # Full workflow: thin orchestrator invoking superpowers at phase boundaries
  references/
    skill-format.md          # Frontmatter schema + size budgets
    anti-patterns.md         # Lint checklist
    skill-authoring-plan-template.md  # Adapts writing-plans' task format for skill authoring
    writing-skills-summary.md   # Writing-skills principles distillation (Phase 3 context)
agents/
  consistency-tester.md      # Opus agent: multirun validation — runs skill-tester N times, compares, collects judgment
  skill-refiner.md           # Opus agent: dual-lens failure analysis, proposes targeted skill edits
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

`plugin.json` points to `./skills/` and `./agents/` — the plugin loader discovers contents by convention.

### Phase → Superpowers Mapping

| Phase | Superpowers Skill | What It Does |
|-------|-------------------|-------------|
| 1. Design | `superpowers:brainstorming` | Collaborative design spec with 2-3 approaches |
| 2. Plan | `superpowers:writing-plans` | Bite-sized tasks using skill-authoring template |
| 3. Build & Test | `skill-creator` (from `anthropics/skills`) | Build skill from plan, eval with baseline comparison, grade, iterate, optimize description |
| 4. Validate | (skill-bench native via consistency-tester) | Multirun testing, user judgment, pattern analysis, refinement |
| 5. Finalize | (skill-bench native) | Lint, CSO check, promote |

### Skill-Creator in Phase 3

Phase 3 delegates to Anthropic's official `skill-creator` skill with structured handoff context (design spec, plan, skill-format constraints, writing-skills principles). Skill-creator handles: write → eval (with-skill + baseline) → grade → viewer → iterate → description optimization.

### Runtime Artifacts (created in user projects, not this repo)

- `.skillbench/config.json` — project config (drafts_dir, evals_dir, test_model, context_files)
- `.skillbench/specs/{skill-name}-design.md` — design specs from Phase 1
- `.skillbench/plans/{skill-name}-plan.md` — implementation plans from Phase 2
- `.skillbench/evals/{skill-name}/evals.json` — eval definitions from Phase 3 (git-tracked)
- `.skillbench/workspace/{skill-name}/` — skill-creator workspace: iterations, grading, benchmarks (gitignored)
- `.skillbench/test-history/{skill-name}/` — consistency-tester results + refinement records (gitignored)
- `.skillbench/test-cases/{skill-name}.json` — test case library for multirun validation
- `skills/drafts/` — default location for in-progress skill drafts

## Conventions

- Skill names must be kebab-case: `^[a-z0-9]+(-[a-z0-9]+)*$`
- Skill descriptions must start with "Use when" or "Use this when"
- SKILL.md files should stay under 200 lines; split heavy content into `references/`
- Token estimate for markdown: `lines * 6`
- The skill-explorer agent is read-only — it must never write files
- The skill-tester, consistency-tester, and skill-refiner agents use model `opus`; skill-explorer uses `haiku`

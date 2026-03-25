# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin for interactive skill authoring. It provides two skill-creation paths — `skill-bench` (5-phase workflow: Design → Plan → Build & Test → Validate → Finalize) and `skill-bench-express` (fast path: Parse → Fetch → Triage → Generate → Present → Sample Run → Finalize) — plus four companion agents (`skill-tester`, `consistency-tester`, `skill-refiner`, `skill-explorer`).

**Requires:** [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin (auto-installed on first use).

Install: `claude plugin marketplace add https://github.com/Yiminnn/skill-bench-plugin && claude plugin install skill-bench`

## Development

No build, test, or lint commands — the entire codebase is markdown files and a JSON plugin manifest. Validation happens at skill-authoring time via the lint pass in Phase 5 (Finalize) of the workflow.

## Architecture

```
.claude-plugin/plugin.json   # Plugin manifest — declares skills/ and agents/ dirs
skills/skill-bench/
  SKILL.md                   # Full workflow: thin orchestrator invoking superpowers at phase boundaries
  references/
    skill-format.md          # Frontmatter schema + size budgets (shared with express)
    anti-patterns.md         # Lint checklist (shared with express)
    skill-authoring-plan-template.md  # Adapts writing-plans' task format for skill authoring
skills/skill-bench-express/
  SKILL.md                   # Express path: resource URLs + prompt → working skill
  references/
    role-extraction-guide.md # Role classification cues + extraction depth rules
agents/
  consistency-tester.md      # Opus agent: multirun validation — runs skill-tester N times, compares, collects judgment
  skill-refiner.md           # Opus agent: dual-lens failure analysis, proposes targeted skill edits
  skill-tester.md            # Opus agent: simulates skill execution, returns structured eval
  skill-explorer.md          # Haiku agent: read-only scanner for drafts and test history
```

`plugin.json` points to `./skills/` and `./agents/` — the plugin loader discovers contents by convention.

### Express Workflow (skill-bench-express)

| Phase | What It Does |
|-------|-------------|
| 1. Parse | Extract skill description + classify resource URLs by role |
| 2. Fetch | Download resources, extract at role-appropriate depth (full/citation-driven/targeted/structural) |
| 3. Triage | Show extraction summary for user confirmation (skippable) |
| 4. Generate | Produce SKILL.md + references + lightweight design spec |
| 5. Present | Hard gate — user reviews draft in editor |
| 6. Sample Run | Optional — single skill-tester invocation; can escalate to consistency-tester |
| 7. Finalize | Lint (shared references) + promote |

No superpowers dependency — uses standard Claude Code tools + shared agents.

### Full Workflow Phase → Superpowers Mapping

| Phase | Superpowers Skill | What It Does |
|-------|-------------------|-------------|
| 1. Design | `superpowers:brainstorming` | Collaborative design spec with 2-3 approaches |
| 2. Plan | `superpowers:writing-plans` | Bite-sized tasks using skill-authoring template |
| 3. Build & Test | `superpowers:subagent-driven-development` | Execute tasks with spec + behavioral review |
| 4. Validate | (skill-bench native via consistency-tester) | Multirun testing, user judgment, pattern analysis, refinement |
| 5. Finalize | (skill-bench native) | Lint, CSO check, promote |

### Reviewer Roles in Phase 3

- **Reviewer 1 (Spec Compliance):** Checks against design spec + `skill-format.md`
- **Reviewer 2 (Behavioral Testing):** Spawns `skill-tester` for simulated execution + pressure tests from writing-skills TDD

### Runtime Artifacts (created in user projects, not this repo)

- `.skillbench/config.json` — project config (drafts_dir, test_model, context_files)
- `.skillbench/specs/{skill-name}-design.md` — design specs from Phase 1
- `.skillbench/plans/{skill-name}-plan.md` — implementation plans from Phase 2
- `.skillbench/test-history/{skill-name}/` — simulated + pressure test results
- `.skillbench/test-cases/{skill-name}.json` — test case library for multirun validation
- `skills/drafts/` — default location for in-progress skill drafts

## Conventions

- Skill names must be kebab-case: `^[a-z0-9]+(-[a-z0-9]+)*$`
- Skill descriptions must start with "Use when" or "Use this when"
- SKILL.md files should stay under 200 lines; split heavy content into `references/`
- Token estimate for markdown: `lines * 6`
- The skill-explorer agent is read-only — it must never write files
- The skill-tester, consistency-tester, and skill-refiner agents use model `opus`; skill-explorer uses `haiku`

# Skill Bench

Interactive skill authoring for Claude Code — create, test, and refine skills through structured conversation.

## Install

```bash
claude plugin marketplace add https://github.com/Yiminnn/skill-bench-plugin
claude plugin install skill-bench
```

## Quick Start

```
/skill-bench
```

| Phase | What Happens | Powered By |
|-------|-------------|------------|
| 1. Design | Brainstorm approaches, produce design spec | `superpowers:brainstorming` |
| 2. Plan | Generate implementation tasks with TDD steps | `superpowers:writing-plans` |
| 3. Build & Test | Build skill, eval with baseline comparison, iterate | `skill-creator` |
| 4. Validate | Multirun consistency testing, user judgment, refinement | `consistency-tester` + `skill-refiner` |
| 5. Finalize | Lint, validate references, promote | built-in |

### Refine an existing skill

Already have a skill? Skip straight to validation:

```
/skill-bench
> refine path/to/my-skill/SKILL.md
```

### Quick Reference

| You want to... | Say... |
|---|---|
| Create a new skill | `/skill-bench` |
| Refine an existing skill | `/skill-bench` then `refine path/to/skill` |
| Approve a step | `y` or `looks good` |
| Edit the draft yourself | Edit in your editor, then say `I edited it` |
| Run quick test | `yes` (when offered sample run) |
| Run thorough testing | `full validation` |
| Mark a run as failed | `run 3 failed — [what went wrong]` |
| Approve proposed fixes | `approve all` or `approve fix 1 and 3` |
| Finish testing | `validation complete` |
| Check existing drafts | `show me my skill drafts` |

## Components

| Component | Type | Model | Purpose |
|-----------|------|-------|---------|
| `skill-bench` | Skill | — | 5-phase workflow orchestrator |
| `skill-tester` | Agent | Opus | Simulates skill execution, returns structured eval with thinking trace |
| `consistency-tester` | Agent | Opus | Multirun validation: run N times, compare, collect judgment, refine |
| `skill-refiner` | Agent | Opus | Dual-lens failure analysis (cross-run + per-run), proposes targeted edits |
| `skill-explorer` | Agent | Haiku | Read-only scanner for drafts and test history |

## Configuration

On first use, creates `.skillbench/config.json` in your project:

```json
{
  "drafts_dir": "skills/drafts",
  "evals_dir": ".skillbench/evals",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

### Artifacts

| Path | Purpose | Tracked? |
|------|---------|----------|
| `.skillbench/config.json` | Project settings | Yes |
| `.skillbench/specs/` | Design specs | Yes |
| `.skillbench/plans/` | Implementation plans | Yes |
| `.skillbench/evals/` | Eval definitions | Yes |
| `.skillbench/test-cases/` | Test case libraries | Yes |
| `.skillbench/workspace/` | Skill-creator iterations | No |
| `.skillbench/test-history/` | Test results and refinements | No |

## Prerequisites

Requires two dependencies (both auto-installed on first use):

- [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) — Phases 1-2 (brainstorming + planning)
- [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) — Phase 3 (build + eval)

Manual install if needed:

```bash
claude plugin install claude-plugins-official/superpowers
```

## License

MIT

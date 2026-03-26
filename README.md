# Skill Bench

A Claude Code plugin for interactive skill authoring. Create, test, and refine Claude Code skills through conversation.

```
/skill-bench
```

Five phases: Design → Plan → Build & Test → Validate → Finalize.

## Install

```bash
claude plugin marketplace add https://github.com/Yiminnn/skill-bench-plugin
claude plugin install skill-bench
```

### Prerequisites

Requires:
- [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin (Phases 1-2)
- [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill (Phase 3)

Both are installed automatically on first use, or manually:

```bash
claude plugin install claude-plugins-official/superpowers
```

## Usage

The workflow:
1. **Design** — Brainstorm approaches, produce a design spec via `superpowers:brainstorming`
2. **Plan** — Generate implementation tasks with TDD steps via `superpowers:writing-plans`
3. **Build & Test** — Build skill from plan, evaluate with baseline comparison, iterate via `skill-creator`
4. **Validate** — Multirun consistency testing: run the skill N times, compare outputs, collect pass/fail judgments, refine based on failure patterns
5. **Finalize** — Lint, validate references, promote to target location

### Validate with multirun testing

The consistency-tester agent runs the skill against real test cases multiple times:

1. Define test cases in `.skillbench/test-cases/{skill-name}.json`
2. The agent runs each case N times (default: 5) via the skill-tester
3. A consistency summary shows what's stable vs. variable across runs
4. You mark each run pass/fail with notes on what's wrong
5. The skill-refiner agent analyzes failure patterns across runs and proposes targeted edits
6. You approve edits, re-run, repeat until satisfied

### Explore existing drafts

```
> Show me my skill drafts
```

The skill-explorer agent scans your drafts directory and reports status.

## Components

| Component | Type | Model | Purpose |
|-----------|------|-------|---------|
| `skill-bench` | Skill | — | 5-phase authoring workflow with superpowers + skill-creator orchestration |
| `skill-tester` | Agent | Opus | Simulates skill execution, returns structured evaluation with thinking trace |
| `consistency-tester` | Agent | Opus | Multirun validation: run collection, consistency analysis, user judgment, refinement loop |
| `skill-refiner` | Agent | Opus | Dual-lens failure analysis (cross-run + per-run), proposes targeted skill edits |
| `skill-explorer` | Agent | Haiku | Read-only scanner for drafts and test history |

## Project Config

On first use, Skill Bench creates `.skillbench/config.json` in your project:

```json
{
  "drafts_dir": "skills/drafts",
  "evals_dir": ".skillbench/evals",
  "test_model": "claude-opus-4-6",
  "context_files": []
}
```

- `drafts_dir` — where draft skills are written (default: `skills/drafts`)
- `evals_dir` — where eval definitions are stored (default: `.skillbench/evals`)
- `test_model` — model for simulated testing (default: `claude-opus-4-6`)
- `context_files` — glob patterns for files to include in test context

### Artifact Locations

| Directory | Purpose | Git Track? |
|-----------|---------|------------|
| `.skillbench/config.json` | Project settings | Yes |
| `.skillbench/specs/` | Design specs (brainstormed or auto-generated) | Yes |
| `.skillbench/plans/` | Implementation plans from Phase 2 | Yes |
| `.skillbench/evals/` | Eval definitions from Phase 3 (skill-creator) | Yes |
| `.skillbench/workspace/` | Skill-creator workspace (iterations, grading, benchmarks) | No (gitignore) |
| `.skillbench/test-cases/` | Test case libraries for multirun validation | Yes |
| `.skillbench/test-history/` | Test results, judgments, refinement records | No (gitignore) |

The workflow will offer to update `.gitignore` on first use.

## License

MIT

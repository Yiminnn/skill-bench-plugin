# Skill Bench

A Claude Code plugin for interactive skill authoring. Create, test, and refine Claude Code skills through conversation.

Two paths to a working skill:

- **`skill-bench`** — Full 5-phase workflow for complex skills: Design → Plan → Build & Test → Validate → Finalize
- **`skill-bench-express`** — Fast path from reference documents + prompt to working skill: Parse → Fetch → Triage → Generate → Present → Finalize

## Install

```bash
claude plugin marketplace add https://github.com/Yiminnn/skill-bench-plugin
claude plugin install skill-bench
```

### Prerequisites

The full workflow (`skill-bench`) requires the [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin. It will be installed automatically on first use, or manually:

```bash
claude plugin install claude-plugins-official/superpowers
```

The express path (`skill-bench-express`) has no plugin dependencies.

## Usage

### Express: Create a skill from reference documents

Best for domain experts who have reference documents and know what the skill should do. Provide a description with embedded URLs and role annotations:

```
/skill-bench-express create a new skill for 361 HCT/P assessment,
here is the FDA guidance document that you should follow step by step
using the decision tree as the main guide for workflow:
https://www.fda.gov/media/109176/download

For the same surgical procedure exemption step, follow the
recommendations in this guidance and make this a separate reference:
https://www.fda.gov/media/89920/download

The regulations that underlie this framework should serve as the
ground truth and be a separate reference:
https://www.ecfr.gov/current/title-21/chapter-I/subchapter-L/part-1271
```

The express path:
1. Classifies each resource by role (primary guide, ground truth, named reference, background)
2. Fetches and extracts content at role-appropriate depth
3. Shows a triage summary for confirmation
4. Generates a SKILL.md + reference files
5. Presents the draft for editor review
6. Optionally runs a sample test or full multirun validation
7. Lints and promotes to final location

### Full workflow: Author a skill from scratch

Best for complex skills that benefit from design exploration and iterative TDD:

```
/skill-bench
```

The full workflow:
1. **Design** — Brainstorm approaches, produce a design spec via `superpowers:brainstorming`
2. **Plan** — Generate implementation tasks with TDD steps via `superpowers:writing-plans`
3. **Build & Test** — Execute tasks with simulated + pressure testing via `superpowers:subagent-driven-development`
4. **Validate** — Multirun consistency testing: run the skill N times, compare outputs, collect pass/fail judgments, refine based on failure patterns
5. **Finalize** — Lint, validate references, promote to target location

### Validate with multirun testing

Available in both paths. The consistency-tester agent runs the skill against real test cases multiple times:

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
| `skill-bench` | Skill | — | Full 5-phase authoring workflow with superpowers orchestration |
| `skill-bench-express` | Skill | — | Fast path: reference documents + prompt to working skill |
| `skill-tester` | Agent | Opus | Simulates skill execution, returns structured evaluation with thinking trace |
| `consistency-tester` | Agent | Opus | Multirun validation: run collection, consistency analysis, user judgment, refinement loop |
| `skill-refiner` | Agent | Opus | Dual-lens failure analysis (cross-run + per-run), proposes targeted skill edits |
| `skill-explorer` | Agent | Haiku | Read-only scanner for drafts and test history |

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
| `.skillbench/specs/` | Design specs (brainstormed or auto-generated) | Yes |
| `.skillbench/plans/` | Implementation plans from Phase 2 | Yes |
| `.skillbench/test-cases/` | Test case libraries for multirun validation | Yes |
| `.skillbench/test-history/` | Test results, judgments, refinement records | No (gitignore) |

The workflow will offer to update `.gitignore` on first use.

## Shared Infrastructure

Both paths share agents and references — no duplication:

| Shared Component | Used By Express | Used By Full Workflow |
|------------------|-----------------|----------------------|
| `skill-format.md` | Generate + Finalize | Phase 1, 3, 5 |
| `anti-patterns.md` | Finalize | Phase 5 |
| `skill-tester` agent | Sample Run | Phase 3 (behavioral testing) |
| `consistency-tester` agent | Escalation from Sample Run | Phase 4 |
| `skill-refiner` agent | Via consistency-tester | Via consistency-tester |
| `.skillbench/` artifacts | Config, specs, test history | All artifacts |

### Recovery Path

A skill started with Express can transition to the full workflow. The auto-generated design spec in `.skillbench/specs/` is compatible with `skill-bench` Phase 2 (Plan), so you can switch to TDD-style development mid-stream.

## License

MIT

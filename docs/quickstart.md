# Quick Start

## Install

```bash
claude plugin marketplace add https://github.com/Yiminnn/skill-bench-plugin
claude plugin install skill-bench
```

## Create a Skill — `/skill-bench`

```
/skill-bench
```

| Phase | Skill/Agent Used | What Happens |
|-------|-----------------|--------------|
| 1. Design | `superpowers:brainstorming` | Brainstorm approaches, produce design spec |
| 2. Plan | `superpowers:writing-plans` | Generate implementation tasks |
| 3. Build & Test | `skill-creator` | Build skill from plan, eval with baseline comparison, iterate |
| 4. Validate | `consistency-tester` + `skill-refiner` | Multirun testing, user judgment, pattern-based refinement |
| 5. Finalize | built-in | Lint + promote |

Requires the [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin and [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) skill (both auto-installed on first use).

## Quick Reference

| You want to... | Say... |
|---|---|
| Create a skill | `/skill-bench` |
| Approve a step | "y" or "looks good" |
| Edit the draft yourself | Edit in your editor, then say "I edited it" |
| Quick test | "yes" (when offered sample run) |
| Thorough testing | "full validation" |
| Mark a run as failed | "run 3 failed — [what went wrong]" |
| Approve proposed fixes | "approve all" or "approve fix 1 and 3" |
| Finish testing | "validation complete" |
| Check existing drafts | "show me my skill drafts" |

# Quick Start

Two ways to create skills: **Express** (from reference documents) or **Full** (from scratch with design exploration).

## Install

```bash
claude plugin install https://github.com/Yiminnn/skill-bench-plugin
```

## Path A: Express (Reference Documents → Skill)

Best when you have source documents and know what the skill should do.

### 1. Create — `/skill-bench-express`

Type one message with what the skill should do and your source documents as URLs. Describe what each document is for — this controls how deeply each one is read.

**Example — regulatory assessment:**
```
/skill-bench-express create a skill for [topic] assessment,
here is the guidance document to follow step by step using the
decision tree as the main guide for workflow: [URL]

For the [sub-topic] step, make this a separate reference: [URL]

The regulations should serve as the ground truth: [URL]
```

**Example — report generation:**
```
/skill-bench-express create a skill that generates [report type],
here is an example of a completed report to use as the template
and main guide: [URL]

Use this style guide as background for formatting: [URL]
```

**Example — process checklist:**
```
/skill-bench-express create a skill for [process name] review,
follow this standard operating procedure step by step as the
main guide: [URL]

These requirements should be the ground truth: [URL]
```

**Role keywords** — how you describe a document determines extraction depth:

| You say... | How it's read |
|---|---|
| "main guide", "step by step", "decision tree" | Full — every branch, criterion, step |
| "ground truth", "regulations", "requirements" | Citation-driven — only sections cited by the main guide |
| "separate reference", "for [topic]" | Targeted — only the named topic |
| "background", "rationale", "context" | Structural — key concepts and definitions only |

### 2. Review triage — automatic (part of `skill-bench-express`)

Claude shows what it extracted from each document. Check that nothing important was missed. Type **y** or tell Claude what's wrong.

### 3. Review draft — automatic (part of `skill-bench-express`)

Claude generates the skill and asks you to review in your editor. Check that the workflow, citations, and terminology are correct. Type **looks good** or describe what to change.

### 4. Test — `skill-tester` / `consistency-tester`

Claude offers a sample run using the **`skill-tester`** agent. Say **yes** for a quick check.

For thorough testing, say **full validation** — Claude spawns the **`consistency-tester`** agent, which runs the skill 5 times per test case, shows what's consistent vs. variable, and you mark each run pass or fail. The **`skill-refiner`** agent analyzes failure patterns and proposes fixes. Re-runs until you're satisfied.

### 5. Finalize — automatic (part of `skill-bench-express`)

Claude lints and promotes the skill to your chosen location.

## Path B: Full Workflow (From Scratch)

Best for complex skills that benefit from design exploration and iterative development. Start with:

```
/skill-bench
```

| Phase | Skill/Agent Used | What Happens |
|-------|-----------------|--------------|
| 1. Design | `superpowers:brainstorming` | Brainstorm approaches, produce design spec |
| 2. Plan | `superpowers:writing-plans` | Generate implementation tasks |
| 3. Build & Test | `superpowers:subagent-driven-development` | Execute tasks with `skill-tester` for behavioral testing |
| 4. Validate | `consistency-tester` + `skill-refiner` | Multirun testing, user judgment, pattern-based refinement |
| 5. Finalize | built-in | Lint + promote |

Requires the [superpowers](https://github.com/anthropics/claude-plugins-official/tree/main/superpowers) plugin (auto-installed on first use).

## Quick Reference

| You want to... | Say... |
|---|---|
| Create from documents | `/skill-bench-express [description + URLs]` |
| Create from scratch | `/skill-bench` |
| Skip triage | "skip" |
| Approve a step | "y" or "looks good" |
| Edit the draft yourself | Edit in your editor, then say "I edited it" |
| Quick test | "yes" (when offered sample run) |
| Thorough testing | "full validation" |
| Mark a run as failed | "run 3 failed — [what went wrong]" |
| Approve proposed fixes | "approve all" or "approve fix 1 and 3" |
| Finish testing | "validation complete" |
| Check existing drafts | "show me my skill drafts" |

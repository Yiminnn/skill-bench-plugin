# Skill-Authoring Plan Template

This template adapts writing-plans' task format for skill authoring. Provide this to writing-plans as context so it produces skill-appropriate tasks instead of code-oriented ones.

## Key Adaptation

Code plans: write test → implement → run test → commit
Skill plans: define behavior → baseline test → write section → simulate → pressure test → commit

This maps to writing-skills' RED → GREEN → REFACTOR cycle:
- Steps 1-3 = **RED** (baseline fails without skill)
- Step 4 = **GREEN** (write minimal skill to pass)
- Steps 5-7 = **REFACTOR** (test, find gaps, close loopholes)

## Task Step Template

Each task in the plan should use this step sequence:

```
### Task N: [Behavioral Capability Name]

**Files:**
- Create/Modify: `{drafts_dir}/{skill-name}/SKILL.md`
- Create (if needed): `{drafts_dir}/{skill-name}/references/{name}.md`

- [ ] **Step 1: Define expected behavior**

What should an agent do when given this skill section?
What should it NOT do?
What rationalizations might it use to skip this behavior?

- [ ] **Step 2: Write baseline pressure scenario**

A prompt that tests this behavior WITHOUT the skill. Must:
- Create realistic pressure (time, complexity, sunk cost)
- NOT mention the skill or its rules
- Be specific enough to observe compliance/violation

[Show exact prompt text]

- [ ] **Step 3: Run baseline test (RED)**

Spawn subagent WITHOUT skill. Give it the pressure scenario. Document:
- What the agent did
- Rationalizations used (verbatim)

Expected: Agent violates or skips the target behavior.

- [ ] **Step 4: Write the skill section (GREEN)**

Write the SKILL.md content addressing violations from Step 3.
[Show exact markdown content]

After writing, report: `SKILL.md: {lines} lines (~{lines * 6} tokens)`

- [ ] **Step 5: Run simulated test**

Spawn the skill-tester agent with current SKILL.md draft + sample input.
Expected: pass or partial.

- [ ] **Step 6: Run pressure test (REFACTOR)**

Spawn fresh subagent WITH skill loaded. Give it scenario from Step 2.
Expected: Agent complies with skill instructions.

- [ ] **Step 7: Iterate if needed**

If Step 5 or 6 failed: identify gap, edit skill section, re-run failing test.

- [ ] **Step 8: Commit**
```

## Task Decomposition

Break skills into tasks by **behavioral capability**, not by markdown heading:

1. Frontmatter + overview (skill identity and trigger)
2. Core workflow (main process the skill teaches)
3. Quality gates / red flags (discipline enforcement)
4. Reference files (if size budget requires splitting)
5. Companion agent (if the skill spawns agents)

Not every skill needs all 5. Simple skills may only need 1-2.

## Size Budget Checkpoints

After each task, check cumulative size:
- SKILL.md < 200 lines: OK
- SKILL.md 200-300 lines: suggest splitting to `references/`
- SKILL.md > 300 lines: strongly recommend splitting

Token estimate: `lines * 6`

## Plan Header

Use this header for skill-authoring plans:

```
# [Skill Name] Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
> to implement this plan. Reviewer 2 runs behavioral testing (simulated execution +
> pressure tests) instead of code quality review.

**Goal:** Create the {skill-name} skill that [one sentence]

**Architecture:** Skill-bench orchestrated. SKILL.md + references/ for heavy content.
Testing via skill-tester agent (simulated) and pressure testing (writing-skills TDD).

**Tech Stack:** Markdown (SKILL.md with YAML frontmatter), Claude Code plugin system
```

## Concurrent Edit Protection

Include this reminder in EVERY task that writes to draft files:

> Before every Write or Edit: read file, compute SHA256, compare to last-known hash.
> If different: show what changed, ask overwrite/incorporate/keep. Update hash after write.

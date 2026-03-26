# Skill-Authoring Plan Template

This template adapts writing-plans' task format for skill authoring. Provide this to writing-plans as context so it produces skill-appropriate tasks instead of code-oriented ones.

## Key Adaptation

Code plans: write test → implement → run test → commit
Skill plans: define behavior → baseline test → write section → simulate → pressure test → commit

This maps to the RED → GREEN → REFACTOR cycle:
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
- Combine at least 2 pressure types for discipline-enforcing skills:
  - **Time:** "the user is waiting", "this is urgent"
  - **Sunk cost:** "you've already written most of the code"
  - **Authority:** "the senior engineer said to skip this"
  - **Exhaustion:** "this is task 7 of 12"
- NOT mention the skill or its rules
- Be specific enough to observe compliance/violation

[Show exact prompt text]

- [ ] **Step 3: Run baseline test (RED)**

Spawn subagent WITHOUT skill. Give it the pressure scenario. Document:

Use the Agent tool to spawn a fresh subagent. Do NOT load the skill being tested — the point is to observe natural behavior without it.
- What the agent did
- Rationalizations used (**record exact verbatim quotes** — these become the rationalization table in the skill)
- Which pressure types triggered which violations

Expected: Agent violates or skips the target behavior.

- [ ] **Step 4: Write the skill section (GREEN)**

Write the SKILL.md content addressing violations from Step 3.
[Show exact markdown content]

After writing, report line count and token estimate (lines multiplied by 6): `SKILL.md: 142 lines (~852 tokens)`

- [ ] **Step 5: Run simulated test**

Spawn the skill-tester agent with current SKILL.md draft + sample input.
Expected: pass or partial.

- [ ] **Step 6: Run pressure test (REFACTOR)**

Spawn fresh subagent WITH skill loaded. Give it scenario from Step 2.
Expected: Agent complies with skill instructions.

- [ ] **Step 7: Iterate if needed**

If Step 5 or 6 failed: identify gap, edit skill section, re-run failing test.
- For each new rationalization discovered, add an explicit counter to the skill
- If 3+ rationalizations target the same rule, build a red flags list
- Close loopholes explicitly — don't just restate the rule, forbid specific workarounds

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

## Skill Type Testing Strategies

Different skill types need different pressure scenarios and test approaches:

| Skill Type | Examples | Test With | Success Criteria |
|-----------|----------|-----------|-----------------|
| **Discipline** | TDD, verification gates | Pressure scenarios with combined pressures + rationalization tables | Agent follows rule under maximum pressure |
| **Technique** | Step-by-step guides, workflows | Application + variation + missing info scenarios | Agent applies technique correctly to new scenarios |
| **Pattern** | Mental models, design patterns | Recognition + application + counter-example scenarios | Agent identifies when/how to apply (and when NOT to) |
| **Reference** | API docs, format specs | Retrieval + application + gap scenarios | Agent finds and correctly uses reference info |

When writing plan tasks, match pressure scenario design to the skill type. Discipline skills need heavy rationalization testing. Reference skills need retrieval accuracy testing.

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

> **For agentic workers:** This plan is executed by the `skill-creator` skill
> during Phase 3 of skill-bench. Skill-creator handles the eval loop:
> write → test (with/without baseline) → grade → viewer → iterate → optimize description.

**Goal:** Create the {skill-name} skill that [one sentence]

**Architecture:** Skill-bench orchestrated. SKILL.md + references/ for heavy content.
Build and eval via skill-creator. Multirun validation via consistency-tester (Phase 4).

**Tech Stack:** Markdown (SKILL.md with YAML frontmatter), Claude Code plugin system
```


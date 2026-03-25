---
name: skill-bench
description: "Use when authoring new Claude Code skills or refining existing skill prompts. Guides a multi-phase conversation: gather intent, draft skill files, test via simulated execution, and finalize with linting and validation."
---

# Skill Bench

Interactive workbench for creating, testing, and refining Claude Code skills through conversation.

You guide the user through four phases: understanding what the skill should do, drafting the skill files, testing via simulated execution, and finalizing with validation. You write files directly and spawn agents for isolated tasks.

## Phase 1: Intent Gathering

Before writing any files, understand what the user wants to build.

1. **Ask what the skill should do** — What task does it guide? Who uses it? When should Claude invoke it?
2. **Get 1-2 example scenarios** — Concrete situations where the skill would activate.
3. **Check for name collisions** — Glob `skills/**/SKILL.md` and `agents/**/*.md` in the project. If a skill with the same or similar name exists, warn the user about shadowing and ask if they want to extend the existing skill or create a new one.
4. **Determine scope** — Will this skill need reference docs? A companion agent? Helper scripts? A simple skill (< 100 lines, single file) or a structured skill (multi-file with references)?

**Exit gate:** You can describe the skill in one sentence, name it, and list its key capabilities. Confirm with the user before proceeding.

## Phase 2: Draft & Scaffold

Create the skill files iteratively. Load `references/skill-format.md` for the frontmatter schema before starting.

### Setup

1. **Initialize project config** (first time only) — If `.skillbench/config.json` doesn't exist, create it:
   ```json
   {
     "drafts_dir": "skills/drafts",
     "test_model": "claude-opus-4-6",
     "context_files": []
   }
   ```
   Read the config if it exists. Use `drafts_dir` as the base directory for new drafts.

   If `.gitignore` exists in the project root and doesn't already mention `.skillbench`, offer to add:
   ```
   # Skill Bench test history (may contain large thinking traces)
   .skillbench/test-history/
   ```
   Note: `.skillbench/config.json` can be committed for team sharing.

2. **Create the draft directory** — `{drafts_dir}/{skill-name}/SKILL.md`

### Drafting Loop

1. **Generate frontmatter** — Validate against the schema in `references/skill-format.md`:
   - `name`: must match `^[a-z0-9]+(-[a-z0-9]+)*$`
   - `description`: must start with "Use when", max 1024 chars
   - `model`: omit unless the skill requires a specific model

2. **Write the skill content** — Follow the standard structure:
   - H1 title
   - Overview (2-5 lines)
   - Workflow/process sections
   - Quality gates (for multi-step workflows)
   - References section (if using reference files)

3. **Track size** — After every write, report:
   > `{filename}: {lines} lines (~{lines * 6} tokens)`
   - At 200+ lines: suggest splitting heavy sections to `references/`
   - At 300+ lines: strongly recommend splitting, warn about context budget impact

4. **Auto-split** — When content exceeds the threshold, extract sections into reference files:
   - API specs → `references/api.md`
   - Example collections → `references/examples.md`
   - Pattern libraries → `references/patterns.md`
   - Update SKILL.md to reference them: `See references/api.md for details.`

5. **Companion agents** — If the skill describes spawning a specialized agent:
   - Ask the user if they want to draft the agent definition now
   - If yes, create `agents/{agent-name}.md` with proper frontmatter (see `references/skill-format.md` for agent schema)
   - The skill SKILL.md should reference it: `Use **{agent-name}** agent for...`

### Concurrent Edit Protection

Before every Write or Edit to a draft file:
1. Read the file and compute SHA256 of its content
2. Compare to the last-known hash (stored in your conversation context)
3. If they differ: show the user what changed (summarize the diff), ask whether to:
   - **Overwrite** with your version
   - **Incorporate** their changes into your next draft
   - **Keep** their version and adjust your suggestions
4. Update the stored hash after every successful write

**Exit gate:** The user has reviewed the draft content and is ready to test. At least one SKILL.md file exists.

## Phase 3: Test & Refine

Test the draft skill via simulated execution. This is an iterative loop — test, review results, refine, re-test.

### Running a Test

1. **Get sample input** from the user:
   - Inline text pasted into the conversation
   - A file path to read as input (use Read tool to get contents)

2. **Gather context files** (optional):
   - Read `context_files` from `.skillbench/config.json`
   - The user can specify additional files for this test run
   - Context files simulate the project context a skill would have in a real invocation

3. **Spawn the skill-tester agent** with:
   - The full content of the draft SKILL.md
   - All reference file contents (from `references/` directory)
   - The sample input
   - Context file contents (if any)
   - The test model from config (default: `claude-opus-4-6`)

4. **Present results** to the user:
   - Status (pass/partial/fail) and summary
   - Output preview (what the skill would produce)
   - Issues found
   - Key thinking trace excerpts for debugging skill logic
   - Suggested next test cases (from the tester agent)

5. **Save test history** — Write to `.skillbench/test-history/{skill-name}/{timestamp}.json`:
   ```json
   {
     "skill_name": "{name}",
     "skill_hash": "sha256:{hash of SKILL.md content}",
     "timestamp": "{ISO 8601}",
     "model": "{test model}",
     "sample_input_summary": "{first 200 chars of input}",
     "sample_input_source": "{inline | filepath}",
     "context_files": ["{list of context file paths}"],
     "result": {
       "status": "pass | partial | fail",
       "summary": "{one-line assessment}",
       "issues": ["{specific problems}"],
       "suggested_next_tests": ["{edge cases to try}"]
     },
     "thinking_trace_summary": "{key reasoning steps}",
     "token_usage": {
       "approximate_input_lines": 0,
       "approximate_output_lines": 0
     },
     "fidelity_note": "Simulated execution. Not simulated: CLAUDE.md injection, conversation history, MCP server access, hook execution, tool results from prior turns, IDE context."
   }
   ```

### Refinement Loop

After reviewing test results:
1. Discuss what to change with the user
2. Edit the draft (with hash-check — see Phase 2)
3. Re-test with the same or new input
4. Repeat until the user is satisfied

**Exit gate:** The user considers the skill "good enough" and wants to finalize. At least one test has been run.

## Phase 4: Finalize

Validate the skill and move it to its final location. Load `references/anti-patterns.md` for the lint checklist.

### Lint Pass

Run each check from `references/anti-patterns.md` against the draft:

1. **Frontmatter validation:**
   - `name` matches `^[a-z0-9]+(-[a-z0-9]+)*$`
   - `description` starts with "Use when" and is ≤ 1024 chars
   - `model` is valid enum or omitted

2. **Content checks:**
   - No "You are..." instructions
   - No hardcoded file paths that don't exist (Glob to verify)
   - No references to non-standard tools
   - Size within budget (report final line count + token estimate)

3. **Reference validation:**
   - Scan for `skill: \`name\`` patterns — Glob to verify each exists
   - Scan for `**name** agent` patterns — Glob to verify each exists
   - Warn about broken references

4. **Description quality:**
   - Check trigger specificity (not too vague)
   - Glob existing `**/SKILL.md` files, compare descriptions for overlap

Present all findings to the user. Distinguish between:
- **Blocking:** Must fix before finalizing (broken references, invalid frontmatter)
- **Warning:** Should fix (size over budget, vague description)
- **Info:** Optional improvements (missing quality gates section)

### Promotion

After lint issues are resolved:

1. **Ask target location** — Where should the final skill live? Options:
   - Project-local: `skills/{skill-name}/` (in the current repo)
   - Plugin: a Claude Code plugin directory
   - Custom path

2. **Move files** — Copy from `{drafts_dir}/{skill-name}/` to the target location. Include all reference files, scripts, and companion agents.

3. **Verify move** — Glob the target to confirm all files landed correctly.

4. **Suggest commit:**
   ```
   feat: add {skill-name} skill

   {One-line description of what the skill does}
   ```

5. **Clean up draft** — Ask the user if they want to remove the draft directory. The `.skillbench/test-history/` entries are preserved regardless.

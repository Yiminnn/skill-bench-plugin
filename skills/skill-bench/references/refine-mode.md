# Refine Mode — Skill Import Guide

## Locating the Skill

Accept the skill path from the user. Three supported source types:

### 1. Explicit Path

User provides a path to a SKILL.md file or its parent directory.
- If path points to a directory, look for `SKILL.md` inside it
- Validate: file exists, has YAML frontmatter with `name` field

### 2. Name Lookup (Drafts)

User names a skill (e.g., "refine my-cool-skill").
- Look in `{drafts_dir}/{skill-name}/SKILL.md`
- If `.skillbench/config.json` exists, read `drafts_dir` from it
- Otherwise default to `skills/drafts`

### 3. Installed/Published Skill

User provides a path under `skills/` (not `skills/drafts/`).
- This is a published skill — do NOT modify in place
- Copy entire directory to `{drafts_dir}/{skill-name}/` (including `references/`, `agents/` subdirs)
- Inform user: "Copied to drafts for safe editing. Original preserved at {path}."

## Validation

After locating the skill:

1. Read the SKILL.md file
2. Parse YAML frontmatter — extract `name`, `description`
3. Verify `name` matches kebab-case pattern: `^[a-z0-9]+(-[a-z0-9]+)*$`
4. Glob for `references/` and note any companion files
5. Report to user:
   - Skill name and description
   - Location (original and draft copy if applicable)
   - Size: `{lines} lines (~{lines * 6} tokens)`
   - References found (list or "none")

## Workspace Setup

1. **Config** — Read or create `.skillbench/config.json` (same as Phase 3 Setup step 1)
2. **Directories** — Create if not exist:
   - `.skillbench/test-cases/`
   - `.skillbench/test-history/{skill-name}/`
3. **Test cases** — Check for `.skillbench/test-cases/{skill-name}.json`
   - If found: "Found existing test cases. Use these or create new ones?"
   - If not found: prompt user to define test cases (same format as Phase 4 Setup)
   - If `{evals_dir}/{skill-name}/evals.json` exists, offer to seed from evals

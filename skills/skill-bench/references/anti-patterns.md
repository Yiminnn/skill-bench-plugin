# Skill Anti-Patterns — Lint Checklist

Use this checklist during Phase 5 (Finalize) to catch common issues before promoting a skill draft.

## Frontmatter Issues

- [ ] **Missing "Use when..." in description** — Description field should start with "Use when" or "Use this when". This is the convention Claude uses to match skills to user intent.
- [ ] **Name not kebab-case** — Must match `^[a-z0-9]+(-[a-z0-9]+)*$`. No underscores, uppercase, or spaces.
- [ ] **Description exceeds 1024 characters** — Truncate or split into a shorter trigger description and in-body details.
- [ ] **Invalid model value** — Must be one of: `opus`, `sonnet`, `haiku`. Omit the field entirely to inherit.

## Content Issues

- [ ] **"You are..." instructions** — Starting instructions with "You are a..." conflicts with Claude's system prompt. Instead, describe the task: "This skill guides..." or "When invoked, this skill...".
- [ ] **Hardcoded file paths that don't exist** — Scan for absolute paths or project-specific paths (e.g., `src/utils/helper.ts`). Skills should use Glob patterns or describe what to look for, not hardcode paths.
- [ ] **References to non-standard tools** — Check any tool name references against the Claude Code toolset: `Read`, `Write`, `Edit`, `Glob`, `Grep`, `Bash`, `Agent`, `Skill`, `WebFetch`, `WebSearch`, `NotebookEdit`. Flag unknown tool names.
- [ ] **SKILL.md exceeds 300 lines without references/** — Large skills should split heavy content into `references/` files. The main SKILL.md is loaded on every invocation.
- [ ] **Broken skill/agent references** — Scan for `skill: \`name\`` and `**name** agent` patterns. Verify each referenced skill/agent exists via Glob in the project's skills/ and agents/ directories.

## Description Quality

- [ ] **Description overlaps with existing skills** — Glob all `**/SKILL.md` files, read their descriptions, check for trigger condition overlap. Two skills that fire on the same scenario cause confusion.
- [ ] **Description is too vague** — Should name specific scenarios or actions, not broad categories. "Use when working with code" is too vague. "Use when authoring new Claude Code skills through iterative conversation and testing" is specific.

## Structural Issues

- [ ] **No overview section** — First section after the H1 title should orient the reader in 2-5 lines.
- [ ] **No quality gates** — Skills that guide multi-step workflows should include verification checkpoints.
- [ ] **Companion agent referenced but not created** — If the skill mentions spawning a specific agent, that agent's `.md` file must exist.

## Eval Coverage

- [ ] **No evals.json** — Skills should have an `evals.json` in `.skillbench/evals/{skill-name}/` with assertions covering the skill's main behavioral capabilities.
- [ ] **Evals don't cover plan capabilities** — Cross-reference eval cases against the implementation plan's behavioral capabilities. Each major capability should have at least one eval case.

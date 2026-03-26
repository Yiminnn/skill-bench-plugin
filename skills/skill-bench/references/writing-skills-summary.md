# Writing-Skills Principles Summary

Lightweight distillation of `superpowers:writing-skills` for context injection during skill-creator handoff. Not the full skill — just the principles that matter at build time.

## The Iron Law

No skill without a failing test first. Write skill before testing → delete, start over. This applies to new skills AND edits to existing skills. No exceptions — not for "simple additions", not for "just adding a section", not for "documentation updates".

## RED-GREEN-REFACTOR for Skills

| Phase | What Happens |
|-------|-------------|
| **RED** | Run pressure scenario WITHOUT skill. Document exact behavior and rationalizations (verbatim). |
| **GREEN** | Write minimal skill addressing those specific violations. Run same scenario WITH skill — agent should comply. |
| **REFACTOR** | Agent found new rationalizations? Add explicit counters. Re-test until bulletproof. |

## CSO (Claude Search Optimization)

**Description = when to use, NOT what the skill does.**

- Start with "Use when..." — trigger conditions only
- Never summarize the skill's workflow in the description
- Third person, specific scenarios, not vague categories
- Descriptions that summarize workflow create a shortcut Claude will take — the skill body becomes documentation Claude skips

## Rationalization Bulletproofing

- Close every loophole explicitly — don't just state the rule, forbid specific workarounds
- Address "spirit vs letter" arguments early: "Violating the letter of the rules is violating the spirit of the rules"
- Build rationalization tables from baseline testing — every excuse agents make goes in the table
- Create red flags lists for easy self-checking when under pressure

## Token Efficiency

- Move implementation details to tool `--help` references
- Use cross-references to other skills instead of repeating content
- Compress examples: one excellent example beats many mediocre ones
- Eliminate redundancy — don't repeat what's in referenced skills

## Generalize Over Overfit

Skills must work across millions of future invocations, not just the test examples. When issues persist after 2+ fix attempts, explore broader metaphors and patterns rather than fiddly narrow changes. Avoid overfitting to specific test case wording.

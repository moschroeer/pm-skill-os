---
name: evaluate-skill
description: Evaluate a skill with structured test cases and a pass/fail recommendation. Use when the user asks to test a skill, score a skill, validate whether a skill is ready, or evaluate a skill after changes.
---

# Evaluate Skill

**ID:** SKILL-META-003
**Owner:** [Your Name]
**Governance:** 🟡 Yellow — pass/fail recommendation gates whether other skills are marked active; owner must confirm the score before a skill is promoted
**Version:** 1.0

## Purpose
Run a structured evaluation against an existing skill, score the outputs against its definition of done, and recommend whether it is ready to use.

## Inputs
- `skill_id` (required): The skill ID, skill name, or exact file path to evaluate.
- `test_mode` (optional): `ai_generated` or `manual`. Default to `ai_generated`.
- `manual_test_cases` (required for `manual`): Test scenarios to run.

## Context To Load
Read the full target skill before doing anything else. If the skill file cannot be found, ask the user for the exact path.

## Workflow
1. Read the target skill and extract its inputs, workflow, definition of done, and failure modes.
2. Stop if the skill is incomplete or still contains placeholders.
3. Generate 3-5 test cases that cover:
   - a normal case
   - an edge case
   - a failure-prone case
4. Show the proposed test cases and let the owner adjust them before running.
5. Run each case by following the skill exactly.
6. After each run, score each definition-of-done item as `Pass`, `Partial`, or `Fail`.
7. Calculate the per-run score with this formula:
   - `Pass = 1`
   - `Partial = 0.5`
   - `Fail = 0`
   - `score = earned_points / total_points * 100`
8. Present the result after each run and let the owner correct the assessment.
9. Compile a final report with:
   - results by run
   - repeated issues
   - recommended fixes
   - final recommendation: `PASS`, `NEEDS FIXES`, or `FAIL`

## Quality Bar
- Do not rubber-stamp a pass.
- Include edge and failure-prone cases, not just easy ones.
- Treat repeated failures as systemic issues.
- A skill is only truly ready if it performs consistently, not just once.

## Definition Of Done
- All planned test cases were executed.
- Each run includes a scored checklist against the target skill's definition of done.
- A final report was produced.
- Repeated issues were called out explicitly.
- The owner reviewed the report before the final recommendation.

## Negative Examples
1. **Rubber-stamp pass:** AI marks all DoD items as Pass without scrutiny. Score is inflated. Skill gets used with hidden defects that surface in production.
2. **Only standard cases tested:** Edge and failure-prone cases skipped. The skill passes on clean inputs but fails on the first unusual real-world input.
3. **No report produced:** Evaluation happens in conversation but nothing is written down. No way to track what changed between versions or why.

## Learning Log
*Append-only. One line per entry.*

- [Date] — [Learning]

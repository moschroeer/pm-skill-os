---
name: [kebab-case-skill-name]
description: [REQUIRED — write as trigger conditions, not a summary. Example: "Apply this rule when the user asks to [do X]. Triggers include: '[phrase 1]', '[phrase 2]', '[phrase 3]'."]
---

# [Skill Name]

**ID:** SKILL-[DEPT]-[NNN]
**Owner:** [Name]
**Governance:** [🟢 Green | 🟡 Yellow | 🔴 Red] — [one-line rationale: who sees the output, how reversible, where the judgment sits. See `CLAUDE.md` for criteria.]
**Version:** 1.0

## Purpose
[2–3 sentences. What does it do, when is it triggered, what does it produce? This is for humans reading the file — Cursor reads the `description` frontmatter field above.]

## Inputs
- `[input_name]` (required / optional): [What it is and where the user gets it.]

## Context To Load
List context files Cursor should make available when this skill runs. Group by tier; cap at 5–7 sources total. Remove tiers you don't use.

**Tier 1 — always load:**
- `context/company/profile.md`
- `context/people/{owner}/communication_style.md`

**Tier 2 — skill-specific:**
- `[path/to/template-or-reference.md]`

**Tier 3 — on demand only:**
- `[path/to/background-source.md]`

## Workflow
1. [Specific instruction. No ambiguity about what to do. If the AI would need to guess, it's not specific enough.]
2. **Run a readiness check** *(include whenever the skill operates on variable input — e.g. a brief, a dataset, a request — and the quality of the output depends on what's in that input)*: before producing the main output, surface a structured check of the key inputs — what is present, what is missing, confidence level, and whether the skill can proceed or needs clarification. Format as a small table or bullet list. Cite the source of each claim inline: *(Source: [file / page / tool / model knowledge — validate with [person]])*.
3. [Step]
4. [Step — 4 to 10 steps total]
5. **⏸ Review gate** *(include if output is external or has brand/quality implications — delete otherwise)*: present current output to the owner; continue only when they confirm.
6. [Continue as needed]
7. Once finished, tell the owner: "To log this run, type `/log-output`."

## Quality Bar
- [Specific quality criterion the AI should hold itself to during execution.]
- [Criterion]
- [Criterion]

## Definition Of Done
- [Specific, binary — passes or it doesn't.]
- [Criterion]
- [Criterion]
- [Criterion]
- [Criterion — minimum 5 items]
- No placeholder text remaining in output.
- Owner confirmed the output before delivery (if review gate applies).

## Negative Examples
1. **[Name of failure mode]:** [Specific description. What does it look like? Why is it wrong? Not "output was bad" — describe the actual failure pattern.]
2. **[Name of failure mode]:** [Description]
3. **[Name of failure mode]:** [Description — minimum 3 examples]

## Learning Log
*Append-only. One line per entry.*

- [Date] — [Learning]

<!--
NOTES FOR SKILL AUTHORS

The `description` field at the top is the most important thing to get right.
Cursor reads it to decide when to attach this skill. If it doesn't match what
a real user would actually type, the skill will never trigger.

Write it as: "Apply this rule when the user asks to [X]. Triggers include:
'[phrase]', '[phrase]', '[phrase]'."

Use phrases your team would actually type — not what you wish they'd type.

Context references only resolve if the files exist at those paths in your
repo. If a file is missing, Cursor proceeds without it and the skill silently
degrades. Verify all referenced files exist before activating the skill.

Run-logging belongs to `/log-output` (SKILL-META-004). Do not embed Notion
API code in this skill — delegate to /log-output and (if needed) /file-feedback.

Remove this comment block before saving the skill.
-->

---
name: create-new-skill
description: Create a new reusable project skill from a recurring workflow. Use when the user asks to create a new skill, document a process as a skill, formalize a recurring task, or turn a workflow into a Cursor skill.
---

# Create New Skill

**ID:** SKILL-META-002
**Owner:** [Your Name]
**Governance:** 🟡 Yellow — produces new governance-bearing artifacts; per `governance.mdc`, structural skill changes require owner approval and re-evaluation before activation
**Version:** 2.0

## Purpose
Turn a recurring workflow into a reusable skill with a clear trigger description, a concrete SOP, tiered context references, a binary definition of done, and named failure modes.

## Inputs
- `task_description` (required): The workflow to turn into a skill.
- `owner_name` (required): The person who owns the process.

## Context To Load
Load these if they exist:

- `.cursor/skills/_template/SKILL.md`
- `context/company/profile.md`
- `context/people/{owner_name}/role_profile.md`

For PM-oriented skills, also load:

- `context/shared/pm_framework.md`
- Your use-case registry, if you keep one

If the owner context files do not exist, ask for the missing company and role context directly instead of inventing it.

For PM-oriented skills, first check whether your use-case registry (if you keep one) already contains a matching brief. If it does, use that entry as the starting brief for the skill and use `context/shared/pm_framework.md` to decide the phase, governance color, and context pattern.

## Workflow
1. Confirm the task is repeated enough to justify a skill. Only proceed if it has already been done at least three times, follows a similar pattern each time, and has a clearly describable output. If any of those are still missing after clarification, stop and say the task is not ready to become a skill yet.
2. Ask the owner to explain the workflow in plain language: the goal, what they do, and what a finished result looks like.
3. Mirror back the workflow and confirm the summary before moving on.
4. Propose a skill name (verb + noun), 3–5 trigger phrases a real user would actually type, primary input, and primary output. Confirm with the owner before continuing.
5. Surface implicit knowledge by asking the owner each of these questions explicitly — do not skip any:
   - "Walk me through the last time you did this. What was the first thing you did?"
   - "Where does it usually go wrong or take longer than expected?"
   - "What do you check before considering it finished?"
   - "Edge cases — situations where you'd handle it differently?"
   - "What information do you always wish you had before starting?"
   - "What would a smart person get wrong on their first attempt?"
6. Draft a 4–10 step SOP from the answers and refine with the owner until each step is specific enough to execute without guessing.
7. Identify context sources and assign each one to a tier:
   - **Tier 1 — always load:** can't start without it
   - **Tier 2 — skill-specific:** needed for this skill only
   - **Tier 3 — on demand:** occasionally useful, load only when the SOP requests it
   Cap at 5–7 sources total. Do not reference files that do not exist unless the workflow explicitly allows asking the owner for that missing context.
8. Define quality criteria by asking:
   - "If this output were perfect, what would be true?"
   - "The 3 most common mistakes that look okay but aren't?"
   - "Anything that, if wrong, makes the whole output unusable?"
   Write a binary definition-of-done checklist with at least five items, plus at least three named negative examples (concrete failure descriptions, not vague warnings).
9. Present the complete skill summary for review before writing the final file. The summary must include the proposed name, trigger phrases, inputs, tiered context sources, SOP, definition of done, and negative examples. Do not finalize until the owner explicitly confirms the workflow is accurate.
10. Save the generated skill as a valid `SKILL.md` file using the structure in `.cursor/skills/_template/SKILL.md`. The output must include YAML front matter (`name` + `description`), an ID/Owner/Version header, a Purpose section, Inputs, tiered Context To Load, Workflow, Quality Bar, Definition Of Done, named Negative Examples, and an empty Learning Log. Save it to `.cursor/skills/{kebab-case-name}/SKILL.md`. Assign the next available `SKILL-{DEPT}-{NNN}` ID by checking the existing `.cursor/skills/` folder or the Skill Registry in Notion.
11. Once saved, tell the owner: "To log this run, type `/log-output`."

## Quality Bar
- Trigger phrases must be phrases a real user would actually type.
- Stop instead of improvising when the task is still too vague or not routine enough.
- SOP steps must be specific enough to execute without guessing.
- Context references should be tiered and as small and relevant as possible.
- PM skills should reuse an existing use-case brief when one exists.
- Do not leave placeholders in the final skill.

## Definition Of Done
- The skill has a valid lowercase kebab-case `name`.
- The description clearly states what it does and when to use it, using realistic trigger phrases.
- Inputs, workflow, tiered context references, and quality checks are all present.
- The skill includes at least five definition-of-done items and at least three named negative examples.
- The output is saved to `.cursor/skills/{kebab-case-name}/SKILL.md` with a unique `SKILL-{DEPT}-{NNN}` ID.
- The new skill's final step delegates run-logging to `/log-output` — no embedded Notion API code.
- No placeholder text remains.
- The owner confirmed the workflow before the file was finalized.

## Negative Examples
1. **Vague description field:** Written as "This skill helps with proposals" — Cursor never triggers it because nothing the user types matches. The description must contain the actual phrases.
2. **Vague workflow steps:** Steps like "process the document" or "write the output." The AI has nothing to execute. Every step must say specifically what to do.
3. **Missing context sources:** The workflow references a template or database but it isn't listed in Context To Load — the AI stalls or hallucinates the content.
4. **Embedded logging logic:** The new skill includes a `fetch()` call to the Notion API in its final step instead of delegating to `/log-output`. Schema changes will require touching every skill — exactly what the split was meant to prevent.

## Learning Log
*Append-only. One line per entry.*

- [Date] — [Learning]

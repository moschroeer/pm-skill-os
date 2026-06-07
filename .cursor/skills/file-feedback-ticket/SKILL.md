---
name: file-feedback-ticket
description: File a structured feedback ticket against a skill into the Feedback DB in Notion. Use when the user says "file feedback", "/file-feedback", "log a bug for [skill]", "this skill needs a fix", "open a feedback ticket", or "track this as an issue".
---

# File Feedback Ticket

**ID:** SKILL-META-005
**Owner:** [Your Name]
**Governance:** 🟢 Green — internal ticket capture in Notion; well-defined fields, easily edited or deleted, no external impact
**Version:** 1.0

## Purpose
Capture a structured issue against an existing skill and write it to the **Feedback DB** in Notion via the MCP. Decouples noticing an issue from fixing it so issues are not lost in chat history.

## Inputs
- `skill_id` (required): The skill the issue applies to (e.g. `SKILL-META-002`).
- `issue_description` (required): One-line summary of what is wrong. Usually given by the user when triggering the skill.
- `output_log_row` (optional): Page ID or title of the Output Log row that surfaced this issue.

## Context To Load
- `notion-schema/NOTION-SCHEMAS.md`

The Notion MCP server `notion` is configured in `.cursor/mcp.json`. All Notion writes go through the MCP.

## Workflow
1. Confirm `skill_id`. If only a skill name was given, search the **Skill Registry** in Notion and confirm the match with the user before continuing.
2. If `issue_description` is unclear, ask: "In one sentence — what's wrong with this skill?" Read the summary back and confirm.
3. Ask for severity and map the answer to one of: `Minor — cosmetic`, `Moderate — affects quality`, `Critical — wrong output`.
4. Ask for type and map the answer to one of: `SOP step unclear`, `Missing context source`, `Wrong trigger phrase`, `DoD gap`, `Context doc stale`, `Other`.
5. Draft a one-paragraph proposed fix that references the specific section of the skill file or context doc that should change. Show it to the user; incorporate corrections until they confirm. The fix must be specific enough to act on without further discussion.
6. Use the Notion MCP to resolve relations:
   - **Skill Registry** page for `skill_id` (`Skill` relation)
   - **Output Log** row matching `output_log_row` if provided (`Source run` relation)
   - **Context KB** page if the fix targets a context document (`Linked to` relation)
   Note any relation that cannot be resolved and continue without that field.
7. Use the Notion MCP to create a page in the **Feedback DB** with:
   - `Name`: the one-line summary
   - `Skill`: relation from step 6
   - `Source run`: relation from step 6, if provided
   - `Severity`: label from step 3
   - `Type`: label from step 4
   - `Status`: `Open`
   - `Reported by`: the current user
   - `Reported date`: today
   - `Proposed fix`: the confirmed draft from step 5
   - `Linked to`: relation from step 6, if applicable
   Leave `Patched by` and `Patched date` empty.
8. Confirm to the user: "Filed. (`[summary]` — `[severity]`, status Open) The owner will see it in the next weekly review." If severity is `Critical — wrong output`, also say: "Flagged as Critical — consider patching before the next run of `[skill_id]`."

## Quality Bar
- Proposed fixes must reference a specific section, never vague advice like "make it clearer".
- Always confirm severity with the user — never auto-classify a usability complaint as `Moderate`.
- Always set the `Skill` relation so weekly review by skill picks up the ticket.
- Never hardcode Notion API keys or database IDs — the MCP resolves databases by name.

## Definition Of Done
- A Feedback DB row exists in Notion with severity, type, and proposed fix populated.
- The `Skill` relation links to the correct Skill Registry row.
- If `output_log_row` was provided, the `Source run` relation is set.
- The proposed fix is concrete enough to act on without further discussion.
- The user confirmed the issue summary and the proposed fix before the row was written.
- `Status` is `Open` and `Reported date` is today.
- No Notion API key or database ID appears in the skill — all writes go through the MCP.

## Negative Examples
1. **Vague proposed fix:** The ticket says "improve the prompt" or "make it clearer." The owner has no idea what to change. Tickets must reference a specific section of the skill or context doc.
2. **Missing relation to Skill Registry:** The Feedback DB row is created but the `Skill` relation is empty. The "By skill" view in the weekly review misses the ticket entirely and the issue is not surfaced to the owner.
3. **Logged without severity confirmation:** The skill auto-classifies a "the output was unusable" complaint as `Moderate` instead of asking. Critical issues get buried in the weekly review queue.

## Learning Log
*Append-only. One line per entry.*

- [Date] — [Learning]

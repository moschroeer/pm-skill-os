---
name: log-skill-output
description: Close out a skill run by capturing time saved and a feedback quality label, then writing an Output Log row in Notion via MCP. Use when the user says "log output", "/log-output", "log this run", "close this skill run", "record time saved", or "I just ran [skill] — log it".
---

# Log Skill Output

**ID:** SKILL-META-004
**Owner:** [Your Name]
**Governance:** 🟢 Green — internal run logging to Notion; structured fields, no subjective judgment, rows are easily corrected
**Version:** 1.2

## Purpose
Centralise run-logging for every skill. Ask two closing questions, derive a feedback quality label, and write one row to the **Output Log** database in Notion via the Notion MCP. Branch to feedback ticketing if the run looks broken.

## Inputs
- `skill_id` (required): ID of the skill that was just run (e.g. `SKILL-META-002`). Ask if not stated.
- `skill_name` (required): Human-readable skill name, used in the Output Log row title.
- `run_by` (required): Person who ran the skill — usually inferable from the active context profile.
- `output_content` (optional): The full final output of the skill run, as markdown. When provided, write it into the body of the Output Log Notion page so other skills can read it back later. Skills that produce reusable artefacts (e.g. weekly project health) should pass this; one-off skills can omit it.

## Context To Load
- `notion-schema/NOTION-SCHEMAS.md`

The Notion MCP server `notion` is configured in `.cursor/mcp.json`. All Notion writes go through the MCP — no API keys or database IDs in this skill.

## Workflow
1. Confirm `skill_id` and `skill_name`. If unclear, ask which skill just ran and search the Skill Registry in Notion for the closest match before proceeding.
2. Check whether fixes were already made during the active session before asking question 1. If the chat history shows that the user requested changes and those were applied within the same session, pre-fill question 1 with a summary of what was changed and propose a quality label (`Minor edits` for small in-session fixes, `Major rework` for substantial restructuring). Ask the user to confirm or correct — do not ask them to re-describe changes they already gave.
3. Ask the user:
   > "Quick closing questions:
   > 1. [Pre-filled summary of in-session fixes, or 'Anything that needed fixing, or a step that felt off?' if no fixes were made] — confirm or correct.
   > 2. Roughly how many minutes did this save you compared to doing it manually?"
   Wait for both answers (or confirmation of the pre-fill).
4. Derive a feedback quality label from the answer to question 1: `Used as-is` (looks good / no changes), `Minor edits` (small fixes applied in-session or by user), `Major rework` (substantial changes), or `Wrong output` (unusable). If ambiguous, propose a label and ask the user to confirm.
5. Use the Notion MCP to search the **Skill Registry** database for a row where the `ID` property equals `skill_id`. Capture the page ID for the `Skill` relation. If no row exists, tell the user the relation will be omitted and continue.
6. **If any skill files were modified during the active session** (including the skill being logged, or any skill updated as a side effect of the run), update their Skill Registry rows using `notion-update-page` with `command: update_properties`. Set: `Version` (from the file header), `date:Last updated:start` (today), `Status` → `"Needs update"` if the version changed (signals re-evaluation may be needed), and `Notes` (one-line summary of what changed). Do this for every modified skill, not just the one being logged. If the skill was a patch-only change (wording, formatting) that does not warrant re-evaluation, keep `Status` as `"Active"`.
7. Use the Notion MCP to create a page in the **Output Log** database with:
   - `Name`: `[skill_name] — [YYYY-MM-DD]`
   - `Date`: today
   - `Skill`: relation to the Skill Registry page from step 4 (omit if not found)
   - `Run by`: person property if `run_by` is in the workspace, otherwise plain text
   - `Time saved (min)`: the number from question 2
   - `Feedback quality`: the label from step 3
   - `Notes`: the verbatim answer to question 1, or `No issues.` if "looks good"
   - `Notes`: the verbatim answer to question 1 (or the confirmed pre-fill), noting whether fixes were applied in-session by the AI or by the user manually
   Do not set `Input summary`, `PM decision`, or `Feedback ticket`.
8. If `output_content` was provided, append it to the body of the newly created Output Log page as markdown blocks via the Notion MCP. Keep it as the page body, not as the `Input summary` property — `Input summary` is reserved for inputs.
9. Confirm to the user: "Logged. (`[skill_name]` — `[date]`, `[time_saved]` min, `[quality_label]`). Skill Registry updated for: [list of skills synced]."
10. If the quality label is `Major rework` or `Wrong output`, prompt: "This looks like a real issue worth tracking. Want me to file a feedback ticket? Type `/file-feedback` to start one." Otherwise end silently.

## Quality Bar
- Never guess a feedback quality label on an ambiguous answer — confirm with the user.
- When fixes were made in-session, pre-fill question 1 with a summary of those changes rather than asking the user to re-describe them.
- Always sync the Skill Registry for every skill modified during the session — not just the one being logged. The registry is the source of truth for version and status.
- Never hardcode Notion API keys or database IDs — the MCP resolves databases by name.
- Always attempt the Skill Registry lookup so rollups remain accurate.
- Surface the `/file-feedback` prompt whenever quality is `Major rework` or `Wrong output`.

## Definition Of Done
- The user answered both closing questions (or confirmed the pre-filled in-session summary).
- A feedback quality label was assigned and confirmed if ambiguous.
- An Output Log row exists in Notion with all required fields populated.
- The `Skill` relation is set, or its absence was reported to the user.
- Every skill file modified during the session has its Skill Registry row updated (Version, Last updated, Status, Notes).
- The confirmation message lists all skills whose registry entries were synced.
- If the label was `Major rework` or `Wrong output`, the user was prompted to run `/file-feedback`.
- If `output_content` was passed, the full markdown is present in the Output Log page body (not in the `Input summary` property).
- No Notion API key or database ID appears in the skill — all writes go through the MCP.

## Negative Examples
1. **Logging without confirmation:** The skill picks `Used as-is` for an ambiguous answer like "mostly fine, a couple of things" without asking. The Output Log fills with mislabelled rows and the ROI/quality views become unreliable.
2. **Skipping the Registry lookup:** The Output Log row is created without the `Skill` relation. The Skill Registry rollups (Runs total, Time saved total) silently undercount and dashboards drift from reality.
3. **Suppressing the feedback prompt:** A run is labelled `Wrong output` but the user is not pointed at `/file-feedback`. The issue stays in chat history and is never converted into a tracked ticket.
4. **Re-asking what was already fixed:** The user made iterative fixes during the session and the skill asks "what needed fixing?" as if none of it happened. This wastes the user's time and misses the chance to pre-fill an accurate label from session context.
5. **Skill Registry drift:** A skill was modified during the run but its registry row still shows the old version number and last-updated date. The registry becomes unreliable as a source of truth for what version is actually in the repo.

## Learning Log
*Append-only. One line per entry.*

- [Date] — [Learning]
- 2026-05-10 - v1.1: added in-session fix detection to step 2 — when the user made iterative changes during the active session, pre-fill question 1 with a summary of those changes and propose the quality label rather than asking the user to re-describe them. Added negative example 4.
- 2026-05-10 - v1.2: added Skill Registry sync step (step 6) — when any skill file was modified during the session, update its registry row (Version, Last updated, Status, Notes) as part of logging. Confirmation message now lists all skills synced. Added negative example 5 (registry drift). Updated Definition of Done and Quality Bar to match.

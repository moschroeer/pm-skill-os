---
name: build-context-profile
description: Build a structured context profile for a person, a team, or the company. Saves files into the matching scope under `context/`. Use when the user asks to build a context profile, onboard a person into the system, set up team context (mission, members, KPIs, roadmap), or capture the company profile.
---

# Build Context Profile

**ID:** SKILL-META-001
**Owner:** [Your Name]
**Governance:** 🟡 Yellow — writes durable context files that shape every downstream skill's behaviour; interviewee must confirm accuracy before files are saved
**Version:** 2.0

## Purpose
Conduct a structured interview and produce Markdown context files in the right scope:
- **Person** → `context/people/{name}/` (4 files)
- **Team** → `context/teams/{team-slug}/` (3 files)
- **Company** → `context/company/profile.md` (1 file)

Skills resolve a person's team via the `team:` field in `context/people/{name}/_meta.md`.

## Inputs
- `entity_type` (required): `person`, `team`, or `company`. If unclear, ask before continuing.
- `entity_name` (required): Person's name, team name, or company name.
- `team_slug` (required for `person` and `team`): Kebab-case slug for the team folder. For a person, this is the team they belong to.
- `session_mode` (optional): `full` or `update`. Default to `full`.
- `doc_to_update` (required for `update`):
  - For `person`: `role`, `priorities`, or `comms`
  - For `team`: `team_profile`, `kpis`, or `roadmap`
  - For `company`: `profile`

## Context To Load
Load whichever templates apply to the entity type:

**Person:**
- `context/template/people/{name}/_meta.md`
- `context/template/people/{name}/role_profile.md`
- `context/template/people/{name}/priorities_and_goals.md`
- `context/template/people/{name}/communication_style.md`

**Team:**
- `context/template/teams/{team-slug}/team_profile.md`
- `context/template/teams/{team-slug}/kpis.md`
- `context/template/teams/{team-slug}/roadmap.md`

**Company:**
- `context/template/company/profile.md`

If the templates do not exist, use the structure documented in `context/template/README.md`.

## Workflow

### Common
1. Confirm `entity_type` and `entity_name`. If type is missing, ask whether the user wants to onboard a person, set up team context, or capture the company profile.
2. Confirm folder name conventions: kebab-case slugs (e.g. `jane-doe`, `platform`). Verify the chosen `team_slug` matches existing folders if relevant.

### If `entity_type = person`
3. Confirm the person's name and the `team_slug` they belong to. Verify `context/teams/{team_slug}/` exists; if not, recommend running this skill in `team` mode first.
4. Interview for **role profile**, summarise, confirm.
5. Interview for **priorities and goals**, summarise, confirm.
6. Interview for **communication style**, including at least one real writing sample, summarise, confirm.
7. Present the full summary and pause for review before writing files.
8. Write:
   - `context/people/{slug}/_meta.md` (frontmatter: `name`, `team`)
   - `context/people/{slug}/role_profile.md`
   - `context/people/{slug}/priorities_and_goals.md`
   - `context/people/{slug}/communication_style.md`

### If `entity_type = team`
3. Confirm `team_slug`. Check whether `context/teams/{team_slug}/` already exists.
4. Interview for **team profile** (mission, ownership/charter, members, cross-functional contacts, ways of working, decisions, pods, current tensions). Summarise, confirm.
5. Interview for **KPIs**: which company KPIs the team is derived from, the team-level metrics, targets, and per-pod ownership. Summarise, confirm.
6. Interview for **roadmap**: this quarter's theme, current initiatives, next up, parked items. Summarise, confirm.
7. Present the full summary and pause for review before writing files.
8. Write:
   - `context/teams/{team_slug}/team_profile.md`
   - `context/teams/{team_slug}/kpis.md`
   - `context/teams/{team_slug}/roadmap.md`

### If `entity_type = company`
3. Interview for **company profile** (what we do, who we serve, positioning, products/services, stage, what we are NOT). Summarise, confirm.
4. Pause for review before writing.
5. Write `context/company/profile.md`.

### Final
9. Add a `Last updated:` line at the top of each written file with today's date.

## Quality Bar
- Use concrete details, not generic filler.
- Preserve the interviewee's own wording where possible.
- Do not write files until the review summary is confirmed.
- For person entities, the comms style file must include a real writing example.
- For team entities, KPIs must explicitly link back to the company KPIs they derive from. Roadmap items must reference the relevant team KPI.
- Folder slugs are kebab-case (no spaces, no umlauts, no capitals).

## Definition Of Done
- All required files for the chosen `entity_type` exist at the correct scope path.
- For `person`: `_meta.md` has both `name:` and `team:` frontmatter fields, and the referenced `context/teams/{team}/` folder exists.
- No placeholder text remains.
- Each file has a `Last updated:` line.
- The interviewee confirmed the summary before files were written.
- For `person`, the communication style file contains a real example.
- For `team`, KPI rows name the company KPI they derive from; roadmap rows link to a team KPI.

## Negative Examples
1. **Wrong scope:** Person context written into `context/teams/...` or team KPIs duplicated into a person folder. Each piece of context lives in exactly one scope — pick the right one before writing.
2. **Too generic:** Files say "works in marketing, focuses on growth" — useless for AI calibration. Concrete details only.
3. **No writing example:** Comms style has rules but no real sample. The AI has no calibration anchor.
4. **Skipped review gate:** Files generated before the interviewee confirmed accuracy. Misunderstandings silently degrade every future execution.
5. **Orphaned person:** A person folder is written with `team: foo` but `context/teams/foo/` doesn't exist. Skills will fail to resolve team-level context. Verify the team folder first.
6. **KPIs without lineage:** Team `kpis.md` lists metrics but doesn't say which company KPI they derive from. The roadmap then can't be tied back to company priorities.

## Learning Log
*Append-only. One line per entry.*

- 2026-05-03 — v2.0: refactor to support three scopes (person / team / company). Person folders now hold 4 files (incl. `_meta.md` pointer); team folders hold 3 (`team_profile`, `kpis`, `roadmap`); company is a single file at `context/company/profile.md`. Added DoD checks for KPI lineage and team-folder existence.

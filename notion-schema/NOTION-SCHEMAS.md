# Notion Database Schemas — haystack-os

Five databases. Each has a purpose, a set of properties, and a relationship to the others.
Set them up in this order: Skill Registry → Context KB → Output Log → Feedback DB → Knowledge Feed.

---

## 1. Skill Registry

**Purpose:** One row per skill. The canonical record of what skills exist, who owns them, and whether they're working.

**Notion database name:** `Skill Registry`

| Property name | Type | Options / notes |
|---|---|---|
| Name | Title | Skill name — matches the filename, e.g. "Build Context Profile" |
| ID | Text | e.g. `SKILL-META-001` — must match the ID in the `SKILL.md` file |
| Status | Select | `Draft`, `Active`, `Needs update`, `Deprecated` |
| Owner | Person | The team member responsible for this skill's quality |
| Trigger phrases | Text | Paste the trigger phrases from the `description` field. Makes it easy to look up which skill handles what without opening the file. |
| File path | URL | Link to the `.cursor/skills/{name}/SKILL.md` file in your repo (GitHub/GitLab URL works well here) |
| Version | Number | Matches the version in the file header. Increment when SOP changes. |
| Last updated | Date | Date the `SKILL.md` file was last changed |
| Last evaluated | Date | Date SKILL-META-003 was last run on this skill |
| Avg DoD score | Number | Most recent evaluation score (0–100). Shows quality at a glance. |
| Notes | Text | Freeform — active limitations, known edge cases, things to fix next |
| Default output destination | Text | Where this skill writes its primary output, e.g. `Notion → {Team} Roadmap`, `Notion → Knowledge Feed (Source: User interview)`, `Repo: context/{scope}/`, `Ephemeral — chat only`. Required for all new skills. The skill's final step uses this to know where to write. |
| Output type | Select | `Artifact` (deliverable: PRD, newsletter, report, file), `Intel` (structured findings → Knowledge Feed), `Meta` (system maintenance: logging, evaluation), `Ephemeral` (chat-only answer) |
| Runs (total) | Rollup | Source: Output Log → Skill relation. Calculate: Count. Add this after Output Log is created. |
| Time saved — total (min) | Rollup | Source: Output Log → Time saved (min). Calculate: Sum. |
| Time saved — avg per run (min) | Rollup | Source: Output Log → Time saved (min). Calculate: Average. Shows whether a skill is saving more or less over time. |
| % used as-is | Formula | `toNumber(prop("Runs (total)")) > 0 ? round(100 * prop("Used as-is runs") / toNumber(prop("Runs (total)"))) : 0` — requires a separate "Used as-is runs" rollup filtering for that quality label. Shows how often a skill's output needs no editing. |

**Views to create:**
- **All skills** — full table, default
- **Active only** — filter: Status = Active
- **Needs attention** — filter: Status = "Needs update" OR Avg DoD score < 80
- **By owner** — group by Owner
- **By output type** — group by Output type. Sanity-check the artifact/intel/meta balance.
- **Missing destination** — filter: Default output destination is empty AND Output type ≠ Ephemeral. Should always be empty; flags skills that need their destination defined.
- **ROI board** — sort by Time saved — total descending. After a few weeks this shows which skills are worth investing in further, and which aren't doing enough work yet.

---

## 2. Context Knowledge Base

**Purpose:** One page per context document. Skills reference these pages rather than copying content — when a page is updated here, every skill that uses it is automatically current.

**Notion database name:** `Context KB`

| Property name | Type | Options / notes |
|---|---|---|
| Name | Title | Document name, e.g. "Company Profile — Anna" or "Offer Template" |
| Type | Select | `Personal — company`, `Personal — role`, `Personal — team`, `Personal — priorities`, `Personal — comms`, `Tier 1 reference`, `Tier 2 template`, `Tier 3 background` |
| Person | Person | For personal context docs — which team member this belongs to. Leave blank for shared docs. |
| Tier | Select | `Tier 1 — always load`, `Tier 2 — skill-specific`, `Tier 3 — on demand` |
| File path | URL | Path in the repo, e.g. `context/anna/01_company_profile.md`. Skills use this path in their `@file` references. |
| Last updated | Date | When the document was last edited |
| Updated by | Person | Who made the last change |
| Stale? | Checkbox | Check this when the document needs a refresh. Cleared when updated. |
| Linked skills | Relation → Skill Registry | Which skills load this document. Makes it easy to see the impact of an update. |

**Views to create:**
- **By person** — group by Person (shows each team member's 5 context docs together)
- **Tier 1 only** — filter: Tier = "Tier 1 — always load"
- **Stale docs** — filter: Stale? = checked (review monthly)
- **All shared docs** — filter: Person is empty

**Monthly maintenance:** Open the "Stale docs" view and update any documents that are out of date. For personal context docs, a quick 15-minute re-run of relevant SKILL-META-001 sections is usually enough.

---

## 3. Output Log

**Purpose:** One row per skill execution. Tracks what was run, what came out, and what the PM decided to do with it. After a month you'll see which skills are being used, which are never touched, and which always need refinement.

**Notion database name:** `Output Log`

| Property name | Type | Options / notes |
|---|---|---|
| Name | Title | Auto-format: `[Skill name] — [Date]`, e.g. "Draft proposal — 2026-04-20" |
| Date | Date | When the skill was run |
| Skill | Relation → Skill Registry | Which skill was used |
| Run by | Person | Which PM triggered it |
| Input summary | Text | Brief description of what was passed in — enough to understand context later |
| Time saved (min) | Number | Written automatically by the skill's final step via Notion API. PM answers one question; no manual entry needed. |
| Artifact link | URL | Direct link to the output the skill produced (Notion page URL, Google Doc URL, repo file URL, Knowledge Feed row, etc.). Written automatically by the skill's final step. Empty only if Artifact location = `Ephemeral`. |
| Artifact location | Select | `Notion DB`, `Notion page`, `Google Doc`, `Repo file`, `Knowledge Feed`, `Slack message`, `Email`, `Ephemeral`. Lets you filter Output Log by where outputs landed. |
| Feedback quality | Select | `Used as-is`, `Minor edits`, `Major rework`, `Wrong output` — derived automatically from the PM's closing answer and written by the skill. |
| PM decision | Select | `Written to Notion`, `Sent externally`, `Used as draft`, `Discarded`, `Refined manually` — optional, PM can fill in after the run if useful. |
| Notes | Text | PM's verbatim feedback. Written automatically by the skill's final step. |
| Feedback ticket | Relation → Feedback DB | Link to a feedback ticket if one was created from this run. |

**Views to create:**
- **Recent runs** — sort by Date descending, default view
- **By skill** — group by Skill (shows execution frequency per skill)
- **Quality issues** — filter: Feedback quality = "Major rework" OR "Wrong output"
- **Time saved this month** — filter: Date is this month, sort by Time saved descending
- **Missing artifact** — filter: Artifact link is empty AND Artifact location ≠ "Ephemeral". Catches skills that ran but didn't log where the output went.
- **By artifact location** — group by Artifact location. Shows where outputs are actually landing vs. where the Skill Registry says they should.

**How this works now:** The PM answers two questions at the end of every skill run — "anything off?" and "how many minutes did this save?" The skill writes the Notion row automatically. No manual logging step.

---

## 4. Feedback DB

**Purpose:** One row per identified issue. Decouples "I noticed something wrong" from "I've fixed it." Prevents issues from getting lost in conversation history.

**Notion database name:** `Feedback DB`

| Property name | Type | Options / notes |
|---|---|---|
| Name | Title | Short description of the issue, e.g. "Proposal tone too formal for SME customers" |
| Skill | Relation → Skill Registry | Which skill this applies to |
| Source run | Relation → Output Log | Which execution surfaced this issue |
| Severity | Select | `Minor — cosmetic`, `Moderate — affects quality`, `Critical — wrong output` |
| Type | Select | `SOP step unclear`, `Missing context source`, `Wrong trigger phrase`, `DoD gap`, `Context doc stale`, `Other` |
| Status | Select | `Open`, `In progress`, `Patched — needs re-eval`, `Closed` |
| Reported by | Person | Who noticed the issue |
| Reported date | Date | |
| Proposed fix | Text | Specific change to make in the skill file or context doc. AI can draft this; human confirms. |
| Patched by | Person | Who made the change |
| Patched date | Date | |
| Linked to | Relation → Context KB | If the fix involves updating a context doc, link it here |

**Views to create:**
- **Open issues** — filter: Status = Open OR "In progress", sort by Severity
- **By skill** — group by Skill (shows which skills have the most issues)
- **Weekly review** — filter: Status = Open, sort by Reported date ascending (oldest first)
- **Critical only** — filter: Severity = Critical

**Weekly review process:** Open the "Open issues" view. For each ticket: write the proposed fix in the Proposed fix field if it's not there yet, make the edit to the `SKILL.md` file, update version number in the file header, set Status → "Patched — needs re-eval", update Last updated in the Skill Registry. If the fix is significant, run SKILL-META-003 before marking the skill Active again.

---

## 5. Knowledge Feed

**Purpose:** One row per *finding* — a dated, sourced piece of intel produced by a collector or synthesizer skill (or pasted manually). Distinct from Context KB, which is evergreen reference. Knowledge Feed is time-ordered and decays; consumer skills (PRD draft, opportunity briefing) query it filtered by theme + recency.

**Notion database name:** `Knowledge Feed`

| Property name | Type | Options / notes |
|---|---|---|
| Name | Title | Auto-format: `[Source] — [Date]`, e.g. "Slack #review — 2026-05-08" or "User interview (Acme) — 2026-04-30" |
| Source | Select | `Slack #review`, `Slack #product`, `User interview`, `Opportunity scan`, `RCA — KPI deep dive`, `Email digest — <name>`, `Manual paste`, `Competitive scan`, `Support tickets`, `Other`. Extend this list as new collector skills are added. |
| As of | Date | When the underlying observation was made (interview date, digest period end, etc.). Not the row creation date. |
| Themes | Multi-select | Aligned to your team's KPIs and roadmap themes. Used by consumer skills to filter relevant rows. Start with a small list (5–10) and grow deliberately. |
| Summary | Text | Short structured findings — bullets preferred. The thing consumer skills actually read. Keep displacive: enough to use without re-fetching the source. |
| Source link | URL | Back-link to the original where possible (Slack thread, Notion call notes, email). Empty for Manual paste of unattributable content. |
| Team | Text | For now, free text (your team slug). Convert to relation once a Teams DB exists. |
| Created by skill | Relation → Skill Registry | Which skill produced this row. Empty for manually-created rows. |
| Created date | Created time | Auto. Used together with `As of` to spot lag between observation and ingestion. |
| Expires | Formula | `dateAdd(prop("As of"), 90, "days")` — flat 90-day default. Override per row when intel is durable (e.g. RCA findings) or shorter-lived (e.g. weekly digest). |
| Superseded by | Relation → Knowledge Feed (self) | Optional. When a newer finding replaces this one, link it. Lets consumer skills follow the chain to current truth. |
| Notes | Text | PM's freeform annotations after reviewing — context, caveats, "this is what changed my mind" |

**Views to create:**
- **Fresh — last 30d** — filter: As of is within last 30 days, sort by As of descending. Default view.
- **By source** — group by Source. Sanity-check that collectors are actually running.
- **By theme** — group by Themes. The lens consumer skills mimic when querying.
- **Expiring soon** — filter: Expires is within next 14 days AND Superseded by is empty. Triggers refresh-or-archive decision.
- **Stale** — filter: Expires is in the past AND Superseded by is empty. Review monthly; either supersede with a fresh row or accept as historical.
- **Manual entries** — filter: Created by skill is empty. Useful for spot-checking quality of human-pasted intel.
- **Per-collector audit** — filter to one Source, sort by Created date. Catches collector skills that have stopped firing.

**How consumer skills use this:** A skill like `prd-drafting` queries `Knowledge Feed` filtered by `Themes ∋ {this PRD's theme}` AND `As of within last 90 days`, sorted by `As of` descending. It cites Source links in the PRD where findings are referenced, and the PRD's Output Log row gets `Artifact location = Notion DB` plus a list of Knowledge Feed row IDs it pulled from (in Notes or a future "Sources used" relation).

**Manual paste workflow:** A `manual-intel-ingest` skill takes raw text + source label + date from the PM, extracts findings, suggests themes, and writes a Knowledge Feed row. This is the path for sources without MCP/API access (private Slack channels, customer call recordings, paywalled newsletters).

**Maintenance:** Quarterly, run a `link-check` pass over `Source link` URLs. Notion pages move and Slack threads get archived; dead source links rot the trail silently.

---

## Relationships summary

```
Skill Registry  ←──────────────── Context KB
      ↑  (linked skills)                (linked docs)
      │
      ├── Output Log (one row per execution)
      │         │
      ├── Feedback DB (one row per issue)
      │         │
      │         └── Context KB (if the fix is a doc update)
      │
      └── Knowledge Feed (one row per finding; "Created by skill")
                │
                ↑ (queried by consumer skills via theme + recency filter,
                   not via a hard relation — keeps the Feed loosely coupled)
```

---

## Setup order

1. Create **Skill Registry** first — everything else relates to it. Include the new `Default output destination` and `Output type` properties from day one.
2. Create **Context KB** — add the relation back to Skill Registry
3. Create **Output Log** — add relations to Skill Registry and Feedback DB (the Feedback DB relation can be added after step 4). Include `Artifact link` and `Artifact location` from day one.
4. Create **Feedback DB** — add relations to Skill Registry, Output Log, and Context KB
5. Go back to Output Log and add the Feedback DB relation
6. Create **Knowledge Feed** — add the `Created by skill` relation to Skill Registry. No hard relation to Output Log; consumer skills cite Feed row IDs in Output Log Notes.

## What to put in on day one

**Skill Registry:** Add one row for each `SKILL.md` file in `.cursor/skills/`. Set Status = Active for the meta-skills if they're working. Fill in `Default output destination` and `Output type` for every row — this is the audit step that catches under-specified skills.

**Context KB:** Add one row for each context file that exists in your repo. The file path column is the most important one — skills depend on it being correct.

**Output Log, Feedback DB, Knowledge Feed:** Leave empty. They fill up naturally as you use the system. The Feed will look bare for the first week; that's expected. It only becomes useful once collector skills (Slack digest, manual ingest, interview synthesis) start firing regularly.

---

## Wiring up the automatic logging

Each skill's final step makes a Notion API call. Three values need to be filled in per skill before it can log automatically:

**1. NOTION_API_KEY**
Create an internal integration at notion.so/my-integrations. Give it read/write access to the Output Log database. Store the secret in a `.env` file at the root of your repo — never hardcode it in a skill file. In Cursor, the skill reads it as an environment variable.

**2. OUTPUT_LOG_DATABASE_ID**
Open the Output Log database in Notion. Copy the URL. The 32-character string between the last `/` and the `?` is the database ID. It looks like: `1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d`. Same ID used in every skill — it never changes.

**3. THIS_SKILL_NOTION_PAGE_ID**
Open the skill's row in the Skill Registry. Copy the page URL. The ID is the 32-character string at the end. This is unique per skill — each `.mdc` file gets the page ID of its own Skill Registry row.

**Where to store these values:**
```
# .env (repo root — add to .gitignore)
NOTION_API_KEY=secret_xxxxxxxxxxxxxxxxxxxx
OUTPUT_LOG_DATABASE_ID=1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d

# Per-skill IDs live directly in each .mdc file's final step block
# Replace the placeholder constant name with the actual ID string
```

**Rollups activate automatically** once the Output Log relation is set up and the first rows start coming in. Go to the Skill Registry, add the three rollup properties (Runs total, Time saved total, Time saved avg), point each one at the Output Log relation, and choose the calculation. Notion does the rest.

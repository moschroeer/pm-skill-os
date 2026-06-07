# Automation Readiness — Prep Work

**Goal:** make your skills *headless-execution-ready* before scaling the skill library, so that flipping on an autonomous runtime later (launchd, GitHub Actions, headless Claude) is a config change — not a 30-skill rewrite.

**Recommendation:** complete this prep **before** adding the remaining skills. Cost now ≈ 2 hours of template + governance + Notion schema changes. Cost later ≈ retrofitting every new skill, inconsistently. The forcing function also improves skill quality today, even while everything still runs in Cursor.

---

## Concept

Three automation levels per skill:

| Level | Meaning | Runtime today | Runtime after Phase 2 |
|---|---|---|---|
| `auto-ready` | Deterministic, self-contained, no clarifying questions, all inputs resolvable from declared sources | Cursor (manual trigger) | Headless cron / event trigger |
| `assisted` | Requires PM in the loop — judgement call, ambiguous input, or review gate | Cursor (manual trigger) | Cursor / web cockpit |
| `tracked` | Human does the work; skill just records, reminds, or formats | Cursor (manual trigger) | Notification / dashboard surface |

The contract: an `auto-ready` skill must run end-to-end without prompting the user. If it can't, it's `assisted`.

---

## Changes needed

### 1. `SKILL.md` template (`.cursor/skills/_template/SKILL.md`)

- [ ] Add `automation_level` to frontmatter (required field, one of `auto-ready | assisted | tracked`)
- [ ] Add **Inputs** section listing every input the skill needs and where it resolves from (Notion DB row, MCP source, context file). For `auto-ready`, every input must be resolvable without asking the user.
- [ ] Add **Output contract** section: structured output shape, target surface (Notion page / HTML file / Slack message), and citation rule. Default: every output line that references a source carries an inline `[src:...]` token. Skills with `output_type: action` are exempt.
- [ ] Add `output_type: content | action` to frontmatter. `content` = produces something a human reads (subject to citation rule). `action` = creates an artifact elsewhere or just logs (exempt from citation rule). Default is `content`.
- [ ] Add **Trigger** section: for `auto-ready`, what fires it (cron expression, event from MCP source, manual). For `assisted`, the trigger phrase.
- [ ] Add **Failure mode** section: what the skill does if a source is unreachable or returns empty. `auto-ready` skills must not fall back to "ask the user."

### 2. Governance rule (`.cursor/rules/governance.mdc`)

- [ ] Add: skills tagged `auto-ready` must not prompt the user mid-run. All inputs must be resolvable from declared sources at start.
- [ ] Add: any skill that produces human-facing content must cite its sources inline using the standard token format. Unsourced output is refused. The only exemption is skills that don't produce readable content (loggers, reminders, action skills that create artifacts elsewhere) — these declare `output_type: action` in frontmatter to opt out.
- [ ] Add: `auto-ready` skills must declare explicit failure behaviour — no silent fallbacks to interactive mode.

### 3. New shared context files (`context/shared/`)

- [ ] **`citation_format.md`** — canonical token shapes per source: `[src:linear-ENG-1234]`, `[src:zendesk-58812]`, `[src:interview-YYYY-MM-DD-name#Lnn]`, `[src:slack-CHANNEL/pTIMESTAMP]`, `[src:notion-PAGEID]`. One token shape per connected MCP source.
- [ ] **`automation_levels.md`** — definitions of the three levels, the contract per level, and examples. Loaded as Tier 1 context so every skill author sees it.

### 4. PM context profile template (`context/template/people/{name}/priorities_and_goals.md`)

Currently prose-only. For `auto-ready` skills to resolve "what does this PM care about" without asking, this file needs a structured block.

- [ ] Add a frontmatter or top-of-file structured section listing:
  - `squads:` — squads the PM owns / contributes to
  - `initiatives:` — current quarter initiatives owned
  - `kpis:` — KPIs on the hook for
  - `watch_topics:` — keywords / themes to surface in digests
- [ ] Keep the prose narrative below for human readers and `assisted` skills.
- [ ] Re-run SKILL-META-001 against existing context profiles to backfill the structured block.

### 5. Notion schema (`notion-schema/NOTION-SCHEMAS.md` + actual Notion DB)

- [ ] Skill Registry: add `Automation level` Select column with options `Auto-ready`, `Assisted`, `Tracked`.
- [ ] Skill Registry: add view **"Auto-ready"** filtered to `Automation level = Auto-ready` and `Status = Active`. This is the list you'd point a runtime at.
- [ ] Output Log: add `Unsourced claims` number column (written by skill final step) so citation drift is visible.
- [ ] Update `NOTION-SCHEMAS.md` to document both columns and the new view.

### 6. Existing skill audit

- [ ] Walk every skill currently in `.cursor/skills/` and assign an honest `automation_level`.
- [ ] Expect most meta-skills to be `assisted` (e.g. SKILL-META-001 build-context-profile needs PM input). SKILL-META-004 log-skill-output is the obvious `auto-ready` candidate.
- [ ] For each `auto-ready` candidate, verify it really has no interactive prompts. If it does, either re-tag as `assisted` or refactor inputs to be source-resolvable.

### 7. `evaluate-skill` (SKILL-META-003) update

- [ ] Add a DoD check: skill respects its declared `automation_level`. `auto-ready` skills evaluated against headless-runnability — does it work end-to-end with no human input?
- [ ] Add a DoD check: for `content` skills, sample N output lines and verify every `[src:...]` token resolves to a real source ID. Track citation traceability % per skill.

---

## Suggested order

1. Write `automation_levels.md` and `citation_format.md` (defines the contract)
2. Update `SKILL.md` template (encodes the contract per skill)
3. Update `governance.mdc` (enforces it at runtime)
4. Update PM context profile template + structured block (unblocks `auto-ready` personalisation)
5. Update Notion Skill Registry schema + view
6. Audit existing skills, tag them
7. Update `evaluate-skill` DoD checks

Steps 1–3 are the load-bearing ones. The rest follow naturally.

---

## Definition of done

- Every `SKILL.md` in the repo has an `automation_level` field
- Skill Registry "Auto-ready" view exists and is non-empty
- Governance rule blocks unsourced synthesis output and mid-run prompting in `auto-ready` skills
- `citation_format.md` and `automation_levels.md` exist in `context/shared/`
- The next new skill is born compliant — no retrofits needed

---

## Out of scope (intentionally deferred)

- Building the actual headless runtime (launchd / GitHub Actions / Claude Code headless). Do that *after* you have at least 2–3 `auto-ready` skills worth running unattended.
- Insight Ledger / team memory (#8 from the earlier review). Validate the personal setup first.
- Decision Log DB. Add once digest skills are stable enough to be worth measuring outcomes on.

# PM Use Case Registry
**Scope:** Shared — PM workflows
**Last updated:** [YYYY-MM-DD]
**Version:** 0.1

> **Example file.** This is a genericized example registry showing the shape of a
> use-case entry. Each section is a ready-made brief that `create-new-skill`
> (SKILL-META-002) reads when turning a recurring workflow into a skill. Delete
> these examples and add your own as you identify repeatable PM tasks.

---

## What this file is for

One section per PM use case. Each section is self-contained: it gives an executing agent everything it needs to understand the workflow, what context to load, and what output to produce.

This file serves two jobs:
- **Skill building** — load when running SKILL-META-002 to create a new PM skill; each section is the brief for one skill.
- **Skill execution** — load the relevant section when executing a PM task to get the trigger definition, input/output spec, and context list.

For how these use cases fit together as a PM operating model, see `context/shared/pm_framework.md`.

---

## Discovery — Evidence

### UC-PM-001 — Interview Insights

**Category:** Discovery — Evidence
**Team / Scope:** Global
**Trigger phrases:** "extract interview insights", "summarize interviews", "pull takeaways from interviews", "analyze interview notes", "what did we learn from interviews"
**What it does:** Extracts evidence-backed takeaways from a single customer interview so raw input becomes a reviewable draft with pain points, wins, feature requests, and visible uncertainty.
**Inputs:** Raw interview notes, transcript, recording summary, or a single Notion interview page
**Expected output:** Structured interview takeaway draft — customer context, primary pain points, key wins, top feature requests, supporting quotes, and uncertainty / source gaps
**Frequency:** Medium — recurring whenever interviews happen
**Context to load:**
- `context/shared/use_case.md`
- Raw interview data (user-provided)
- Glossary if available

**Skill file:** *(not yet built)*
**Status:** Example — no skill yet

---

## Discovery — Synthesis

### UC-PM-002 — Problem & Opportunity Briefing

**Category:** Discovery — Synthesis
**Team / Scope:** {Team}
**Trigger phrases:** "build betting table brief", "problem and opportunity briefing", "opportunity brief", "what changed since the last brief", "roadmap opportunity"
**What it does:** Consolidates KPI movement, customer evidence, and team context into a structured opportunity brief or delta update that helps decide whether an opportunity should inform a PRD or a prioritisation decision.
**Inputs:** KPI signals, downstream analysis, customer evidence, team context, current priorities, optional existing brief
**Expected output:** Structured opportunity brief — current thesis, evidence summary, what changed since the last brief, KPI levers, roadmap alignment, scope options, and open questions
**Frequency:** Low to medium — run when meaningful new signal suggests the framing may need to change
**Context to load:**
- `context/teams/{team}/team_profile.md`
- `context/teams/{team}/kpis.md`
- `context/teams/{team}/roadmap.md`
- `context/people/{name}/priorities_and_goals.md`

**Skill file:** *(not yet built)*
**Status:** Example — no skill yet

---

## Comms

### UC-PM-003 — Release / "What's New" Notes

**Category:** Comms
**Team / Scope:** Global
**Trigger phrases:** "draft a what's new", "release notes for [feature]", "in-app update copy", "what's new candidates this fortnight"
**What it does:** Drafts short customer-facing release notes for a shipped feature, calibrated to the team's communication style. External-facing, so always reviewed before publishing.
**Inputs:** Feature reference or draft, target surface, optional languages/targeting
**Expected output:** Publish-ready release note — headline, body copy, targeting/notes, with any machine-assisted translations flagged for review
**Frequency:** Medium — release-driven
**Context to load:**
- `context/company/profile.md`
- `context/people/{name}/communication_style.md`
- Feature draft or release reference (user-provided)

**Skill file:** *(not yet built)*
**Status:** Example — no skill yet

---

*Shared context reference — load the relevant section when executing a PM task, or load the full file when building a new PM skill via SKILL-META-002.*

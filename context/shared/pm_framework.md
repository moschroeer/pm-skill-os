# PM Operating Framework
**Scope:** Shared — applies to all PM use cases
**Last updated:** [YYYY-MM-DD]
**Version:** 1.0

> **Example file.** This is a genericized example of a shared context document.
> It captures a reusable conceptual model for PM work. Adapt the wording, phases,
> and governance defaults to your own org, then point `create-new-skill` at it.

---

## What this file is for

This is the conceptual model behind how PM work flows. It explains the five phases of the PM operating loop, how they connect, and what kind of AI assistance is appropriate at each phase.

Load this when you need to understand where a task fits in the bigger picture — especially before building a new PM skill, or when a task spans multiple phases and the right scope isn't obvious.

For the specific use cases and their execution details, see `context/shared/use_case.md`.

---

## The five-phase PM operating loop

PM work follows a repeatable flow from raw signal to shipped communication. The phases are:

```
Evidence → Synthesis → Scoping → Delivery → Comms
```

These aren't sequential in a strict start-to-finish sense — a PM runs all five phases simultaneously across different projects. But any given task belongs to one primary phase, and knowing which phase it's in tells you what the AI should do and which context to load.

---

### Phase 1 — Discovery: Evidence

**What happens here:** Gathering raw signals — customer interviews, support/Slack signal reviews, KPI movement analysis. The goal is to build a factual base before drawing conclusions.

**AI's role:** Extract, organize, and surface patterns from raw material. The PM provides the inputs; the AI structures what's already there. The AI should not synthesize or recommend here — it should surface.

**Failure mode:** Skipping this phase and going straight to synthesis. Output built on thin evidence tends to miss the actual problem and produces solutions that feel right but aren't grounded.

**Context pattern:** Data-heavy, relatively context-light. The raw input matters more than personal context.

**Typical tasks:** interview insights, customer-signal review, KPI/metric deep dive.

---

### Phase 2 — Discovery: Synthesis

**What happens here:** Evidence gathered in Phase 1 gets turned into a structured opportunity framing. This is where signals become a coherent brief for a decision — e.g. for a betting table or prioritisation forum.

**AI's role:** Consolidate, prioritize, and structure. The PM brings judgment on what matters; the AI ensures nothing is lost in translation and the framing is internally consistent. The AI should present options and tensions, not pick the winner.

**Failure mode:** Synthesizing before enough evidence is in — or letting one loud signal dominate the brief while quieter but more important signals get dropped.

**Context pattern:** Balanced — needs both team context (to judge relevance) and priorities (to frame impact correctly).

**Typical tasks:** problem & opportunity briefing for a betting/prioritisation decision.

---

### Phase 3 — Scoping

**What happens here:** An approved opportunity gets turned into a concrete scope. A PRD is written, refined, and aligned with engineering and stakeholders before work begins.

**AI's role:** Structure and draft. The PM owns the direction and the judgment calls; the AI handles the document work, ensures completeness, and flags open questions it can't answer on its own. The AI should not invent scope or make product decisions — it should surface gaps for the PM to decide.

**Failure mode:** Scoping before synthesis is locked — writing a PRD for a problem that isn't fully understood yet. Or leaving the PRD as an eternal "draft" with no clear review gate.

**Context pattern:** Broadest context load of any phase. Needs company, role, team, and priorities to write a PRD that's actually grounded.

**Typical tasks:** PRD drafting, refinement, and publishing.

---

### Phase 4 — Delivery Coordination

**What happens here:** Active work is in flight. The PM's job is to keep progress visible, catch blockers early, and maintain scope alignment across a cycle/sprint.

**AI's role:** Compile and organize what's in flight from multiple sources (issue tracker, chat, to-dos, standup notes) into a clear working picture the PM can act on directly. The output should always include actionable items, not just status.

**Failure mode:** A health brief that describes what's happening but doesn't surface what the PM should do next. Or letting cycle health reporting become a ritual with no follow-through.

**Context pattern:** Team context and priorities are the essential load. Company and role context are rarely needed here.

**Typical tasks:** weekly project health / risk & scope check, daily & weekly planning + attention management.

---

### Phase 5 — Comms

**What happens here:** Work that's been done gets communicated — internally to the team via sprint roundups, to customers via in-app release notes, or to a wider audience via a newsletter.

**AI's role:** Draft to spec. The PM owns the judgment about what to include and what the message is; the AI handles the draft, format adherence, and tone calibration against the communication style file. For external-facing outputs this phase is always 🟡 Yellow governance — PM reviews before anything goes out.

**Failure mode:** Writing comms before the underlying work is clear, producing copy that's technically accurate but doesn't land. Or tone drift from the established communication style, especially when templates aren't loaded.

**Context pattern:** Style-heavy. Communication style and templates matter more here than company strategy context.

**Typical tasks:** sprint roundup, release/"what's new" notes, newsletter drafting, cycle commitment announcement.

---

## Context dependency model

Every PM use case draws from the same small set of context layers. The phase a task belongs to predicts which layers it needs.

| Layer | File | When needed |
|-------|------|-------------|
| Company context | `context/company/profile.md` | Tasks with external framing, product scope, or positioning |
| Role context | `context/people/{name}/role_profile.md` | Planning, attention management, scope ownership |
| Team context | `context/teams/{team}/team_profile.md` | Anything involving coordination, handoffs, or stakeholders |
| Priorities / KPIs | `context/people/{name}/priorities_and_goals.md`, `context/teams/{team}/kpis.md` | Planning, synthesis, and delivery tasks |
| Communication style | `context/people/{name}/communication_style.md` | Any output going to an audience |
| Data sources | User-provided | Evidence phase — raw notes, dashboards, tickets |
| Templates / glossary | Shared or user-provided | Comms and scoping tasks |

No single task needs every layer. Evidence tasks are data-heavy and context-light. Comms tasks are style-heavy and data-light. Scoping tasks need the broadest load.

---

## Governance defaults by phase

The traffic light color for a PM skill follows from where its output goes, which is largely determined by phase. See `.cursor/rules/governance.mdc` (Cursor) or `CLAUDE.md` (Claude) for the full criteria.

| Phase | Default governance | Reason |
|-------|-------------------|--------|
| Evidence | 🟢 Green | Output is internal signal extraction — easily correctable |
| Synthesis | 🟢 Green | Output is an internal brief — reviewed by PM before use |
| Scoping | 🟡 Yellow | PRDs go to engineering and stakeholders — quality review required |
| Delivery | 🟢 Green | Planning output is internal and immediately actionable |
| Comms (internal) | 🟢 Green | Sprint roundups stay inside the team |
| Comms (external) | 🟡 Yellow | Newsletter and release notes go to customers — always requires PM review |

---

## Context update loop

Skill outputs can make context files stale. The pattern for keeping context current without letting AI overwrite it automatically:

**When a skill output implies a context change, the skill proposes a diff — the PM approves — then the change is written to both the repo file and the matching Notion Context KB page.**

### Which kinds of skill outputs trigger which context files

| Skill type | Trigger condition | Context file to update |
|-------|------------------|------------------------|
| Cycle commitment / planning | A new cycle commits new bets | `context/teams/{team}/roadmap.md` |
| Opportunity briefing | An opportunity moves into active planning | `context/people/{name}/priorities_and_goals.md` |
| Project health | Items newly shipped or scope drifts materially | `context/teams/{team}/roadmap.md` |
| KPI deep dive / RCA | A baseline is officially recalculated | `context/teams/{team}/kpis.md` |
| PRD drafting | A bet is formally added to the roadmap | `context/teams/{team}/roadmap.md` |
| `build-context-profile` (update mode) | Explicit intent — always writes | whichever file the user targets |

### The context sync step

Skills that trigger context updates include a closing step labelled **"Context sync"** that:
1. Identifies which context file(s) the output implies should change.
2. Proposes the specific edit as a diff (what to remove, what to add, where).
3. Pauses for the PM to approve. If declined, skips silently.
4. On approval: writes the repo `.md` file and updates the matching Notion Context KB page via MCP.
5. Updates the `Last updated` date in the file header and the matching property in Notion.

### Governance

The context sync step is **always Green governance** — it only writes after explicit PM approval. No skill may write context files autonomously. If a skill produces an output that implies a context change but the PM is not present to approve, the skill notes the suggested update and the PM applies it in the next session.

### Monthly hygiene

`context/people/{name}/priorities_and_goals.md` should be reviewed at the start of each month regardless of whether a skill triggered an update — treat it as a standing task, not just a reactive one.

---

## How to use this framework when building skills

When `create-new-skill` (SKILL-META-002) is building a new PM skill:

1. **Identify the phase** — which of the five phases does this task belong to? That determines the governance color and context load pattern.
2. **Check `context/shared/use_case.md`** — is there already a `UC-PM-*` entry? Use it as the brief. If not, add one before building the skill.
3. **Map the context layers** — use the table above to decide which files go in the skill's context map.
4. **Set governance color** using the defaults above, then adjust if the specific output destination overrides the default.
5. **Write trigger phrases** that match what a PM would actually type — the use case entry is a starting point, but refine with the skill owner.

---

*Shared context reference — load when building a PM skill or when a task spans multiple phases and the operating model needs to be understood before execution.*

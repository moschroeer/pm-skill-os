# haystack-os — a skill-based PM operating system

A starter kit for running your product-management (or any knowledge) work as a library of
reusable, governed **skills**. It is **platform-agnostic**: it ships with native setups for
both **Cursor** and **Claude** (Claude Code / Claude desktop / Claude Cowork), backed by
**Notion** as the versioning and human-in-the-loop layer.

This repo contains only the *operating system* — the meta-skills that build and maintain the
system, the context scaffolding, the governance rules, and the Notion schema. It deliberately
contains **no** company- or person-specific content. You bring that by running the skills.

---

## What's in the box

| Piece | What it is |
|-------|-----------|
| **Meta-skills** (`SKILL-META-001`…`005`) | The skills that run the system: build context profiles, create new skills, evaluate skills, log runs, file feedback. |
| **`_template/SKILL.md`** | The canonical skill template every new skill is generated from. |
| **Governance** | A traffic-light (🟢🟡🔴) rule applied before every skill run. |
| **Context scaffolding** | A three-scope model (company / teams / people) plus a worked, genericized PM example. |
| **Notion schema** | Five-database setup that tracks skills, context, runs, feedback, and intel. |

---

## Platform-agnostic by design

The skill format (`SKILL.md` with `name` + `description` frontmatter) is identical across
tools, so the OS ships as two parallel trees:

| | Cursor | Claude (Code / app / Cowork) |
|---|--------|------------------------------|
| Skills | `.cursor/skills/<name>/SKILL.md` | `.claude/skills/<name>/SKILL.md` |
| Always-on governance | `.cursor/rules/governance.mdc` | `CLAUDE.md` |
| MCP config | `.cursor/mcp.json` | `.mcp.json` |

The two skill trees are mirrors. **Pick the tool you use and you're ready** — or keep both
in sync if your team uses both. (If you only use one, you can safely delete the other tree.)

---

## Directory structure

```
haystack-os/
│
├── CLAUDE.md                            ← Always-on governance + bootstrap (Claude)
├── .mcp.json                            ← MCP servers (Claude)
│
├── .cursor/
│   ├── mcp.json                         ← MCP servers (Cursor)
│   ├── rules/governance.mdc             ← Always-applied governance (Cursor)
│   └── skills/                          ← Canonical skill source (Cursor)
│       ├── _template/SKILL.md
│       ├── build-context-profile/       ← SKILL-META-001
│       ├── create-new-skill/            ← SKILL-META-002
│       ├── evaluate-skill/              ← SKILL-META-003
│       ├── log-skill-output/            ← SKILL-META-004
│       └── file-feedback-ticket/        ← SKILL-META-005
│
├── .claude/skills/                      ← Mirror of the skills above (Claude)
│
├── context/
│   ├── shared/
│   │   ├── pm_framework.md              ← Example: PM operating model + governance defaults
│   │   └── use_case.md                  ← Example: use-case registry for skill building
│   └── template/                        ← Copy these into the live scopes below
│       ├── company/profile.md
│       ├── teams/{team-slug}/           ← team_profile.md · kpis.md · roadmap.md
│       └── people/{name}/               ← _meta.md · role_profile.md · priorities_and_goals.md · communication_style.md
│
├── notion-schema/NOTION-SCHEMAS.md      ← Notion database setup instructions
└── automation-readiness.md             ← Optional: prep for headless/autonomous runs
```

The live `context/company/`, `context/teams/`, and `context/people/` folders are intentionally
absent — `build-context-profile` (SKILL-META-001) creates them from the templates as you
onboard your company, teams, and people.

---

## Quickstart

1. **Clone the repo** and open it in Cursor or Claude.
2. **Connect Notion** (optional but recommended): the Notion MCP is preconfigured in
   `.mcp.json` / `.cursor/mcp.json`. Authenticate when prompted. Then build the five databases
   in `notion-schema/NOTION-SCHEMAS.md`.
3. **Build your context** — run `build-context-profile` (SKILL-META-001) to capture your
   company, then each team, then each person. These files calibrate every other skill.
4. **Create your first skill** — run `create-new-skill` (SKILL-META-002) on a task you do
   often. It interviews you and writes a new `SKILL.md`.
5. **Evaluate and activate** — run `evaluate-skill` (SKILL-META-003), then mark it Active in
   the Notion Skill Registry.
6. **Repeat** for each recurring task. Log runs with `log-skill-output`, file issues with
   `file-feedback-ticket`.

### Running a skill

In Cursor, type the skill's trigger phrase (from its `description`). In Claude, do the same —
`CLAUDE.md` instructs the model to load the matching `.claude/skills/<name>/SKILL.md`, pull
only its declared context, and follow the workflow, pausing at any review gate.

---

## The feedback + versioning loop

```
Skill runs → issue found → Notion feedback ticket
→ owner approves → skill file updated → version bumped
→ re-evaluation → passes → Notion status = Active
```

See `notion-schema/NOTION-SCHEMAS.md` for the full setup.

---

## Skill ID convention

```
SKILL-[DEPT]-[NNN]

DEPT codes:
  META  — system meta-skills
  PM    — product management skills
  [XX]  — add your own dept codes

NNN = sequential 3-digit number per department
```

Examples: `SKILL-META-001`, `SKILL-PM-003`, `SKILL-MKT-001`

---

## Notes

- `context/shared/pm_framework.md` and `use_case.md` are **examples** — adapt or replace them.
- No secrets in the repo: Notion access goes through MCP. If a skill ever needs an API key,
  put it in `.env` (already git-ignored), never in a skill file.
- `automation-readiness.md` is optional reading for when you later want skills to run headless.

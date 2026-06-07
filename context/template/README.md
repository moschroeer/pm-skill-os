# Context Profile Template
*Three scopes: company, teams, people. Copy each template into its live location and fill via the context interview skill.*

---

## File structure

```
context/
  company/
    profile.md                          ← Single source for the org

  teams/{team-slug}/
    team_profile.md                     ← Mission, ownership, members, ways of working, decisions
    kpis.md                             ← Team KPIs derived from company KPIs
    roadmap.md                          ← Quarterly theme, current bets, next up

  people/{name}/
    _meta.md                            ← Pointer to the person's team (frontmatter: name + team)
    role_profile.md                     ← Always loaded
    priorities_and_goals.md             ← Loaded for planning / prioritisation
    communication_style.md              ← Always loaded
```

The `team:` field in `_meta.md` resolves which team folder a person belongs to. A new PM joining an existing team only needs a new `people/{name}/` folder.

---

## Loading rules (for skill context maps)

| Scope | File | When to load |
|-------|------|-------------|
| Company | `company/profile.md` | Always |
| Person | `people/{name}/role_profile.md` | Always |
| Person | `people/{name}/communication_style.md` | Always |
| Person | `people/{name}/priorities_and_goals.md` | Planning, prioritisation, project-related decisions |
| Team | `teams/{team}/team_profile.md` | Coordination, handoffs, escalation, team identity |
| Team | `teams/{team}/kpis.md` | KPI movement, baselines, briefing, sprint roundup, pod-level health |
| Team | `teams/{team}/roadmap.md` | Briefing, PRD scoping, sprint planning, sprint roundup |

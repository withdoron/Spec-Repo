# Repo Reorg Audit — Spec-Repo + private

> Hyphae audit — 2026-04-29
> Scope: Inventory both repos, propose target locations under the new `platform/ • cross-space/ • spaces/` structure, flag ambiguous items.
> Status: AUDIT — moves not yet executed. Awaiting Doron decisions on flagged items.

---

## Executive Summary

| Repo | Markdown files today | Files moving | Files staying | Folders created |
|------|----------------------|--------------|----------------|-----------------|
| Spec-Repo | 47 | 42 | 5 (README, archetypes/, context/) | platform/, cross-space/, spaces/{field-service,team,finance,property-management,meal-prep}/, plus audits/ subfolders |
| private  | 79 | 77 | 2 (README, users/bari/) | platform/, cross-space/, spaces/{field-service,team,finance,property-management}/, concepts/, users/doron/ |

**Net effect:** Both repos take on parallel `platform/ • cross-space/ • spaces/` shape. Existing `audits/`, `checklists/`, `archetypes/`, `context/`, `schemas/`, `users/` folders are preserved or split into the new structure where natural.

**Key decisions surfaced (need Doron's call before moves run):**
1. **Folder name `team` vs. `playmaker`** for the sports/play workspace space (existing folder in Spec-Repo is `spaces/playmaker/`; spec target says `spaces/team/`).
2. **Frequency Station's home** — it's a "community music space," not really a sports/team thing. Goes under `team/playmaker`, or its own space?
3. **Concepts/seedlings folder** — ~5 docs in `private` (BOMB-SQUAD-*, EDUCATION-FUNDING-RESEARCH, FIELD-INSTRUMENT-SEED, TRIAL-PREP-APP-SPEC) don't fit `platform/cross-space/spaces`. Add a top-level `concepts/` (or `seedlings/`) bucket?
4. **Meal Prep space** — `MEAL-PREP-READINESS-AUDIT.md` references it; workspace registry has it as proposed-not-built. Create `spaces/meal-prep/` placeholder, or wait?
5. **CONVERSATION-STARTER.md (Spec-Repo)** — a pure pointer file marked deprecated 2026-02-11 (says "content moved to system prompt"). Keep, archive, or delete?
6. **SILVER-BARTER-CONCEPT.md (private)** — self-marks as superseded by `SILVER-DIME-LAYER.md`. Keep alongside, archive, or delete?

Rest of the calls I made myself; details inline below. Each flagged item is also called out with `⚠️` in the inventory tables.

---

## Target Structure Adaptations

The morning spec proposed:

```
platform/  cross-space/  spaces/{field-service, team, finance, property-management, ...}
```

I adapted in three places:

1. **Kept `archetypes/` at top level (Spec-Repo).** Archetype docs (dog-walker, donut-shop, mechanic, etc.) are pre-space — node concepts not yet promoted to spaces. Promoting them all to `spaces/<archetype>/` would create a flood of empty spaces. Better to keep them clustered.
2. **Kept `context/` at top level (Spec-Repo).** ACTIVE-CONTEXT, PROJECT-BRAIN, SESSION-LOG, SHIP-IT-PROMPT are session-tracking meta-docs — not platform specs. They're operational, not strategic.
3. **Added `platform/decisions/` and `platform/journals/` subfolders (private).** DEC-*.md files cluster (3 of them); HYPHAE/MYCELIA reflection + dated architecture conversations cluster (4 of them). Subfoldering makes `platform/` browseable.

Also:
4. **`platform/audits/`** for full-platform audits (DRIFT-REPORT-2026-03-30) that aren't space-specific.
5. **`spaces/<space>/audits/`** for space-scoped audits (FIELD-SERVICE-* audits, plus the migration phase reports for field-service).
6. **`users/` stays at top level** in private (parallel to spaces/), because it's user-scoped, not platform-strategic.

---

## Inventory — Spec-Repo

### Top-level files → mostly `platform/`

| Current path | Target path | Reason |
|---|---|---|
| README.md | (stay) | Repo entry point |
| ARCHITECTURE.md | platform/ARCHITECTURE.md | Core technical patterns |
| ADMIN-ARCHITECTURE.md | platform/ADMIN-ARCHITECTURE.md | Admin panel structure |
| BUILD-PROTOCOL.md | platform/BUILD-PROTOCOL.md | Universal 16-phase build sequence |
| BUSINESS-DASHBOARD-v2.md | platform/BUSINESS-DASHBOARD-v2.md | Generic business dashboard (all tiers, not field-service-specific) |
| CONCERN-WORKFLOW.md | platform/CONCERN-WORKFLOW.md | Concern triage applies to all entities |
| CONFIG-SYSTEM.md | platform/CONFIG-SYSTEM.md | PlatformConfig entity |
| CONVERSATION-STARTER.md | ⚠️ FLAGGED | Pure pointer file ("content moved to system prompt") — keep/archive/delete? |
| DECISIONS.md | platform/DECISIONS.md | Decision log |
| ENTITY-SYSTEM.md | platform/ENTITY-SYSTEM.md | Archetype + dashboard system |
| EVENT-SCHEMA-VERIFICATION.md | platform/EVENT-SCHEMA-VERIFICATION.md | Event entity field-mapping reference |
| JOBS-MIGRATION-PLAN.md | spaces/field-service/JOBS-MIGRATION-PLAN.md | Field-service-to-jobs namespace rename |
| LOCALLANE-CORE-ARCHITECTURE-v4-1.md | platform/LOCALLANE-CORE-ARCHITECTURE-v4-1.md | v4.1 entity-based architecture |
| MAP-VIEW.md | platform/MAP-VIEW.md | Network page map view (directory feature, not field-service) |
| MISSION-VISION-VALUES.md | platform/MISSION-VISION-VALUES.md | Brand/philosophical foundation |
| MYLANE.md | platform/MYLANE.md | User dashboard spec |
| OWNERSHIP-BADGES.md | platform/OWNERSHIP-BADGES.md | Local ownership classification |
| PHASE-4-MIGRATION-PLAN.md | platform/PHASE-4-MIGRATION-PLAN.md | v4.1 migration plan |
| RECOMMENDATION-SYSTEM.md | platform/RECOMMENDATION-SYSTEM.md | Nod/Story/Vouch system |
| SEEDLING-TRACKER.md | platform/SEEDLING-TRACKER.md | Project health tracker |
| STATUS-TRACKER.md | platform/STATUS-TRACKER.md | Session log + roadmap |
| STEP-1.1-PLATFORM-CONFIG-ENTITY.md | platform/STEP-1.1-PLATFORM-CONFIG-ENTITY.md | PlatformConfig technical assessment |
| STRIPE-CONNECT.md | platform/STRIPE-CONNECT.md | Payment infrastructure |
| STYLE-GUIDE.md | platform/STYLE-GUIDE.md | Gold Standard design system |
| TIER-SYSTEM.md | platform/TIER-SYSTEM.md | Three-tier business model |
| USER-TIERS.md | platform/USER-TIERS.md | User tier definitions |
| WORKFLOW.md | platform/WORKFLOW.md | Development workflow |

### archetypes/ → stay at top level

| Current path | Target path | Reason |
|---|---|---|
| archetypes/archive/_template.md | (stay) | Already archived |
| archetypes/community-org-engine.md | (stay) | Phase 0 archetype |
| archetypes/dog-walker-node.md | (stay) | Phase 0 archetype |
| archetypes/donut-shop-node.md | (stay) | Phase 0 archetype |
| archetypes/event-node.md | (stay) | Phase 0 archetype, decision pending |
| archetypes/fitness-recess-node.md | (stay) | Phase 0 archetype |
| archetypes/house-cleaner-node.md | (stay) | Phase 0 archetype |
| archetypes/landscaper-node.md | (stay) | Phase 0 archetype |
| archetypes/mechanic-node.md | (stay) | Phase 0 archetype |

### context/ → stay at top level

| Current path | Target path | Reason |
|---|---|---|
| context/ACTIVE-CONTEXT.md | (stay) | Session-tracking meta |
| context/PROJECT-BRAIN.md | (stay) | Universal AI briefing |
| context/SESSION-LOG.md | (stay) | Append-only session history |
| context/SHIP-IT-PROMPT.md | (stay) | Session-end SOP |

### audits/ → split by scope

| Current path | Target path | Reason |
|---|---|---|
| audits/DRIFT-REPORT-2026-03-30.md | platform/audits/DRIFT-REPORT-2026-03-30.md | Full-platform drift catalog |
| audits/FIELD-SERVICE-CURRENT-STATE-AUDIT.md | spaces/field-service/audits/CURRENT-STATE-AUDIT.md | Renamed (drop redundant prefix) |
| audits/FIELD-SERVICE-TOGGLE-AUDIT.md | spaces/field-service/audits/TOGGLE-AUDIT.md | Renamed (drop redundant prefix) |
| audits/MEAL-PREP-READINESS-AUDIT.md | ⚠️ spaces/meal-prep/audits/READINESS-AUDIT.md (FLAGGED) | Meal Prep space doesn't exist yet — create placeholder? |

### checklists/ → split

| Current path | Target path | Reason |
|---|---|---|
| checklists/LAUNCH-CHECKLIST.md | platform/checklists/LAUNCH-CHECKLIST.md | Master cross-platform launch tracker |
| checklists/farm-program-launch.md | spaces/field-service/farm-program-launch.md | Phase 1 farm program is field-service-shaped |

### schemas/

| Current path | Target path | Reason |
|---|---|---|
| schemas/PlatformConfig.schema.json | platform/schemas/PlatformConfig.schema.json | Schema for platform-level config entity |

### spaces/

| Current path | Target path | Reason |
|---|---|---|
| spaces/playmaker/PLAYMAKER-ROLES-AND-ONBOARDING-SPEC.md | ⚠️ spaces/team/PLAYMAKER-ROLES-AND-ONBOARDING-SPEC.md (FLAGGED) | Folder name `playmaker` → `team`? |

---

## Inventory — private

### Top-level → mostly `platform/`

#### Organism / agent / platform substrate

| Current path | Target path |
|---|---|
| README.md | (stay) |
| AGENT-SCOPED-QUERY-SPEC.md | platform/ |
| AGENT-WRITE-AND-MYLANE-CONSOLE-SPEC.md | platform/ |
| BOOK-FRAMEWORKS.md | platform/ |
| CATEGORY-ARCHITECTURE-SPEC.md | platform/ |
| COMMUNITY-PASS.md | platform/ |
| DARK-UNTIL-EXPLORED.md | platform/ |
| DASHBOARD-WORKSPACES.md | platform/ |
| DECISIONS.md | platform/ |
| FINANCIAL-ENGINE.md | platform/ |
| JOY-COINS-SPEC.md | platform/ |
| LEGAL-CONSULTATION-PREP.md | platform/ |
| LEGAL-RESEARCH.md | platform/ |
| LIVING-DIRECTORY-SPEC.md | platform/ |
| LIVING-MAP-SPEC.md | platform/ |
| MARKETING-DECK.md | platform/ |
| MICROBUSINESS-KIT.md | platform/ |
| MYCELIA-MCP-SERVER.md | platform/ |
| MYLANE-CONDUCTOR-SPEC.md | platform/ |
| MYLANE-DYNAMIC-LAYOUT.md | platform/ |
| NODE-BOOTSTRAP.md | platform/ |
| NODE-LAB-MODEL.md | platform/ |
| NODE-PLAYBOOK.md | platform/ |
| OPEN-GARDEN-SPEC.md | platform/ |
| ORGANISM-AGENT-TEAM.md | platform/ |
| ORGANISM-ARCHITECTURE-EVOLUTION.md | platform/ |
| ORGANISM-CONCEPT.md | platform/ |
| PRICING-ECONOMICS.md | platform/ |
| RENDERER-AGENT-SPEC.md | platform/ |
| ROLE-ARCHITECTURE.md | platform/ |
| SEEDLING-TRACKER.md | platform/ |
| STATUS-TRACKER.md | platform/ |
| SUPERAGENT-SPEC.md | platform/ |
| TECH-DEBT.md | platform/ |
| THE-GARDEN.md | platform/ |
| docs/NETWORK-EVENTS-SPEC.md | platform/NETWORK-EVENTS-SPEC.md (drop docs/ prefix) |

#### Decisions cluster → `platform/decisions/`

| Current path | Target path |
|---|---|
| DEC-049-CONSOLIDATION-STRATEGY.md | platform/decisions/DEC-049-CONSOLIDATION-STRATEGY.md |
| DEC-056-EVENT-SYNC.md | platform/decisions/DEC-056-EVENT-SYNC.md |
| DEC-060-LIVING-DIRECTORY.md | platform/decisions/DEC-060-LIVING-DIRECTORY.md |

#### Journals/reflections → `platform/journals/`

| Current path | Target path |
|---|---|
| HYPHAE-ARCHITECTURE-2026-03-30.md | platform/journals/HYPHAE-ARCHITECTURE-2026-03-30.md |
| HYPHAE-ARCHITECTURE-2026-03-31.md | platform/journals/HYPHAE-ARCHITECTURE-2026-03-31.md |
| HYPHAE-REFLECTION.md | platform/journals/HYPHAE-REFLECTION.md |
| MYCELIA-REFLECTION.md | platform/journals/MYCELIA-REFLECTION.md |

#### Field service

| Current path | Target path |
|---|---|
| BUSINESS-SCHEDULE-SPEC.md | spaces/field-service/ |
| CONTRACTOR-DAILY.md | spaces/field-service/ |
| FIELD-SERVICE-ENGINE.md | spaces/field-service/ |
| FIELD-SERVICE-WORKSPACE-SPEC.md | spaces/field-service/ |
| BACKFILL-DRY-RUN-2026-04-22.md | spaces/field-service/audits/ |
| MANIFEST-2026-04-22.md | spaces/field-service/audits/ |
| PHASE-2-DRY-RUN-2026-04-23.md | spaces/field-service/audits/ |
| PHASE-2-POSTMIGRATION-REPORT-2026-04-23.md | spaces/field-service/audits/ |

#### Finance

| Current path | Target path |
|---|---|
| FINANCE-WORKSPACE-SPEC.md | spaces/finance/ |
| PERSONAL-FINANCE-NODE.md | spaces/finance/ |

#### Property management

| Current path | Target path |
|---|---|
| PROPERTY-MANAGEMENT-WORKSPACE-SPEC.md | spaces/property-management/ |
| RENTAL-MANAGEMENT.md | spaces/property-management/ |
| DEC-PP-013-ROLE-EVOLUTION.md | spaces/property-management/decisions/DEC-PP-013-ROLE-EVOLUTION.md |

#### Team / Playmaker (⚠️ flagged folder name)

| Current path | Target path |
|---|---|
| BJJ-RANKED-QUEUE.md | ⚠️ spaces/team/ — or own space? |
| FREQUENCY-STATION-SPEC.md | ⚠️ spaces/team/ — or own space? |
| LEAGUE-AND-PLAY-LIBRARY-SEEDLING.md | spaces/team/ |
| PLAY-RENDERER-SPEC.md | spaces/team/ |
| PLAY-TRAINER-SPEC-v2.md | spaces/team/ |
| TEAM-VISIBILITY-ARCHITECTURE.md | spaces/team/ |

#### Cross-space

| Current path | Target path |
|---|---|
| HARVEST-MARKET-STOREFRONT.md | cross-space/ (Harvest involves directory + Joy Coins + Finance) |
| LEARNING-BRANCH-CONCEPT.md | cross-space/ |
| MEAL-PLANNING-CONCEPT.md | cross-space/ (Harvest + Finance) |
| NONPROFIT-LANDSCAPE-EUGENE.md | cross-space/ |
| SILVER-DIME-LAYER.md | cross-space/ |
| SOVEREIGN-WORKER-NETWORK.md | cross-space/ |
| THE-CAMEL-ACADEMY.md | cross-space/ |
| USER-GROUPS-v1.md | cross-space/ |

#### Concepts / seedlings (⚠️ flagged — folder needs to exist)

| Current path | Target path |
|---|---|
| BOMB-SQUAD-GAME-DESIGN.md | ⚠️ concepts/ |
| BOMB-SQUAD-GROWERS.md | ⚠️ concepts/ |
| EDUCATION-FUNDING-RESEARCH.md | ⚠️ concepts/ |
| FIELD-INSTRUMENT-SEED.md | ⚠️ concepts/ (could also be platform — it's organism philosophy) |
| TRIAL-PREP-APP-SPEC.md | ⚠️ concepts/ — or users/doron/ (tied to personal case)? |
| SILVER-BARTER-CONCEPT.md | ⚠️ FLAGGED — superseded by SILVER-DIME-LAYER, keep/archive/delete? |

#### User-scoped

| Current path | Target path |
|---|---|
| TRIAL-PREP.md | users/doron/TRIAL-PREP.md |
| users/bari/BARI-WORK-LIST.md | (stay) |
| users/bari/ENGAGEMENTS-DESIGN-NOTES.md | (stay) |

---

## Flagged items (need Doron's call)

### 1. Folder name: `team` vs `playmaker`
- Existing: `Spec-Repo/spaces/playmaker/`
- Spec target this morning: `spaces/team/`
- Workspace registry code value: `team` (per `community-node/src/config/workspaceTypes.js`)
- The roles spec inside the `playmaker/` folder talks about "team spaces" internally
- **Recommendation:** Rename to `team/`. It's the workspace_type, the spec target, and what the doc itself uses internally. "Playmaker" lives on as customer-facing brand.
- **Affects:** ~7 file moves (Spec-Repo's PLAYMAKER-ROLES-... + 6 private docs going there)

### 2. Frequency Station's space affiliation
- `FREQUENCY-STATION-SPEC.md` is "community music space — Pip-Boy radio model, Studio vs Stage." Not sports.
- Other "playmaker"-tagged docs are sports/coaching (Play Trainer, BJJ, Play Renderer, League).
- **Recommendation:** Drop into `spaces/team/` for now (it's the closest existing space and Frequency Station is still pre-build). Easy to promote later if it earns its own space.

### 3. Concepts / seedlings folder
- 5 docs (BOMB-SQUAD-GAME-DESIGN, BOMB-SQUAD-GROWERS, EDUCATION-FUNDING-RESEARCH, FIELD-INSTRUMENT-SEED, TRIAL-PREP-APP-SPEC) are early ideas without a clear space.
- They don't fit `platform` (not whole-platform) or `cross-space` (not bridging spaces) or any specific space.
- **Recommendation:** Add a top-level `concepts/` folder in `private` alongside `platform/`, `cross-space/`, `spaces/`. It's where seedlings live before they earn promotion to a space.

### 4. Meal Prep space placeholder
- `MEAL-PREP-READINESS-AUDIT.md` exists in Spec-Repo. The audit explicitly says meal_prep is **not** registered in `workspaceTypes.js` yet.
- **Recommendation:** Create `Spec-Repo/spaces/meal-prep/audits/` with the readiness audit and a placeholder README marking the space as "proposed, not yet built."

### 5. CONVERSATION-STARTER.md
- Self-marked deprecated 2026-02-11. Pure pointer ("content moved to Claude system prompt").
- **Recommendation:** Move to `platform/` and let it sit (won't get in the way) — OR delete if you'd rather declutter. Spec said "don't delete, surface for decision."

### 6. SILVER-BARTER-CONCEPT.md
- Self-marks as superseded by `SILVER-DIME-LAYER.md` (2026-02-08).
- **Recommendation:** Move to `cross-space/` alongside SILVER-DIME-LAYER. Future me reading these together is the right context.

---

## Naming collisions & duplicates

### Cross-repo same-name files (NOT collisions — different repos)
- `DECISIONS.md` exists in both repos (different content — public vs. private decisions)
- `STATUS-TRACKER.md` exists in both (different content)
- `SEEDLING-TRACKER.md` exists in both (different content)
- `README.md` in both

These are fine. Same name, different repo, different content.

### Within-repo collisions
None detected. After renaming `audits/FIELD-SERVICE-CURRENT-STATE-AUDIT.md` → `spaces/field-service/audits/CURRENT-STATE-AUDIT.md` and similar, no two files would land at the same path.

---

## Deprecated / superseded candidates

| File | Status | Recommendation |
|---|---|---|
| Spec-Repo/CONVERSATION-STARTER.md | Self-marked deprecated 2026-02-11 | ⚠️ Keep at platform/, or delete. Doron's call. |
| Spec-Repo/archetypes/archive/_template.md | Already archived (in archive/) | Stay where it is |
| private/SILVER-BARTER-CONCEPT.md | Self-marked superseded by SILVER-DIME-LAYER | ⚠️ Keep at cross-space/, or delete. Doron's call. |

No others self-flag as deprecated. The dated journals/reflections (HYPHAE-*, MYCELIA-REFLECTION, MANIFEST-2026-04-22, BACKFILL-DRY-RUN-2026-04-22, PHASE-2-* reports) are historical records, not deprecated.

---

## Content NOT moving

- `Spec-Repo/.gitignore` — root config
- `Spec-Repo/README.md` — repo entry point
- `Spec-Repo/archetypes/*` — staying as `archetypes/` cluster
- `Spec-Repo/context/*` — staying as `context/` cluster
- `private/README.md` — repo entry point
- `private/users/bari/*` — already in correct location

---

## Constraints honored

- All moves use `git mv` (history preserved).
- No file content modified during reorg (filenames may change for redundant prefixes like `FIELD-SERVICE-CURRENT-STATE-AUDIT.md` → `CURRENT-STATE-AUDIT.md`, since the parent folder now carries the space name — flagging this as a near-content change).
- No deletions. Items I'd consider stale are surfaced above.
- Two main commits (Spec-Repo + private), plus a third small commit relocating this audit.

---

## Open questions for Doron — short version

1. ⚠️ Rename `spaces/playmaker/` to `spaces/team/`?
2. ⚠️ Frequency Station — into `team/` for now, or own space?
3. ⚠️ Add `concepts/` folder to private for seedling docs?
4. ⚠️ Create `spaces/meal-prep/` placeholder for the meal-prep audit?
5. ⚠️ CONVERSATION-STARTER.md — keep or delete?
6. ⚠️ SILVER-BARTER-CONCEPT.md — keep or delete?

Once I have answers, the moves run (~115 file moves total, two commits), then this audit relocates.

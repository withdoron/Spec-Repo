# Meal Prep Readiness Audit

> Hyphae audit — 2026-03-31
> Scope: Full platform state + Meal Prep workspace pre-build verification

## Corrected Repo Paths

| Repo | Path | Notes |
|------|------|-------|
| community-node | `~/Documents/GitHub/community-node/` | Codebase |
| Spec-Repo | `~/Documents/GitHub/Spec-Repo/` | Capital S, capital R |
| private | `~/Documents/GitHub/private/` | Lowercase. Contains MEAL-PLANNING-CONCEPT.md |
| ~~locallane-spec-repo~~ | ~~`~/Documents/LocalLane/locallane-spec-repo/`~~ | Deleted 2026-03-31. Canonical repo is `Spec-Repo/`. |

---

## V1: Workspace Registry State

**File:** `community-node/src/config/workspaceTypes.js`

`meal_prep` is **NOT registered**. Five workspace types currently exist:

| ID | Label | Available | Testing Mode | CreateWizard |
|----|-------|-----------|-------------|--------------|
| business | Business | true | — | null (BusinessOnboarding) |
| team | Team | true | — | TeamOnboarding |
| finance | Finance | true | — | FinanceOnboarding |
| fieldservice | Field Service | true | false | FieldServiceOnboarding |
| property_management | Property Management | true | true | PropertyManagementOnboarding |

**Action needed:** Register `meal_prep` workspace type with `available: true, testingMode: true`.

---

## V2: Warm Entry Messages

**File:** `community-node/src/config/warmEntryMessages.js`

Three entries exist: `team`, `business`, `finance`. No `fieldservice`, `property_management`, or `meal_prep`.

**Gap:** Field Service and Property Management are missing warm entry messages. They predate the pattern.

**Action needed:** Add `meal_prep` entry. Consider backfilling `fieldservice` and `property_management`.

---

## V3: MealPrepProfile Entity Search

**Result: NOTHING EXISTS in the codebase.** Zero references to MealPrepProfile, Recipe, RecipeIngredient. No `src/components/mealprep/` directory. No meal-prep references in any server functions.

**However:** `MEAL-PLANNING-CONCEPT.md` exists in the private repo (`~/Documents/GitHub/private/`), dated 2026-02-01. It contains a comprehensive vision document with:
- Doron's original insight about meal planning tied to farmers market + sale ads
- Detailed data model for Recipe entity (very close to what we'll build)
- Five-phase build sequence: Recipe Book → Meal Planner → Market Integration → Sale Ad Scanning → Smart Sourcing
- Tier placement (free: 10 recipes + single-recipe shopping list; member: full planning + sourcing)
- Competitive landscape analysis

The concept doc aligns well with the current workspace pattern. Phase 1 (Recipe Book) maps cleanly to a new workspace type.

---

## V4: Existing Workspace Pattern (Reference: Field Service)

### Profile Entity Pattern
**Entity:** `FieldServiceProfile` — `user_id`, `workspace_name`, `invite_code`, workspace-specific fields, `subscription_tier` (default "full"), `tier_trial_start`, `guide_dismissed`.
**On create:** `FieldServiceOnboarding.jsx:96-117`

### Registration in workspaceTypes.js
Standard block: `{ id, label, icon (string), description, archetypeSupport: false, networkAffinity, available, testingMode, createWizard, roles, tabs[] }`

### MyLane Card Grid
**File:** `src/config/myLaneRegistry.js` — each workspace registers cards with `{ id, label, space, icon, CardComponent, getProfile, timeAware, urgencyWindow, urgencyEntity }`

### Onboarding
Multi-step wizard → `Entity.create()` → navigate to MyLane

### Workspace Guide
**File:** `src/config/workspaceGuides.js` — welcome message + ordered steps with `{ id, title, description, actionLabel, targetTab, icon }`

### Agent Wiring
Each workspace agent uses `agentScopedQuery` for reads, `agentScopedWrite` for writes (tier-gated).

---

## V5: Server Function Inventory

### In repo (`base44/functions/`):
- `getMyLaneProfiles/entry.ts` — queries: FinancialProfile, FieldServiceProfile, PMPropertyProfile, TeamMember
- `agentScopedWrite/entry.ts` — 3-gate enforcement, 22 whitelisted entities across 5 workspaces

### NOT in repo:
- `agentScopedQuery` — exists only in Base44 dashboard. Must be updated there, not via Hyphae.

### agentScopedWrite Mappings to extend:

**WORKSPACE_PROFILE_MAP** — add: `'meal-prep': { entity: 'MealPrepProfile', userField: 'user_id' }`

**ENTITY_WHITELIST** — add: `'meal-prep': ['Recipe', 'RecipeIngredient']`

**ENTITY_FK** — add:
```
Recipe:           { fkField: 'profile_id', workspace: 'meal-prep' },
RecipeIngredient: { fkField: 'profile_id', workspace: 'meal-prep' },
```

---

## V6: Document Staleness

| Doc | Location | Status |
|-----|----------|--------|
| ACTIVE-CONTEXT.md | Spec-Repo `context/` + community-node `context/` | Current (2026-03-31). Were synced. Now updated with Meal Prep priority. |
| STATUS-TRACKER.md | Spec-Repo (source of truth) | More current than community-node copy. Community-node copy updated this session. |
| SEEDLING-TRACKER.md | Spec-Repo (source of truth) | More current than community-node copy. Both now updated with Meal Prep seedling. |
| SESSION-LOG.md | Spec-Repo `context/` + community-node `context/` | Both current through 2026-03-31. Community-node copy has audit session appended. |
| MEAL-PLANNING-CONCEPT.md | `private/` repo | Exists, dated 2026-02-01. Vision-level spec. |

**Key finding:** Spec-Repo (`~/Documents/GitHub/Spec-Repo/`) has the most up-to-date copies of STATUS-TRACKER.md and SEEDLING-TRACKER.md. Community-node copies were staler but have been updated this session.

---

## V7: Spec Repo State

- `MEAL-PLANNING-CONCEPT.md` — exists in **private** repo (not Spec-Repo)
- `MEAL-PREP-SEEDLING.md` — does NOT exist anywhere. Not needed — the concept doc in private/ serves this purpose.
- Spec-Repo `context/ACTIVE-CONTEXT.md` — exists and is synced with community-node
- Spec-Repo `context/SESSION-LOG.md` — exists and is comprehensive

---

## What Already Exists for Meal Prep

| What | Where | Status |
|------|-------|--------|
| Vision concept doc | `private/MEAL-PLANNING-CONCEPT.md` | Complete (2026-02-01) |
| Entity schemas | Nowhere | Not created |
| Codebase references | Nowhere | Zero references |
| Config entries | Nowhere | Not registered |
| Server function mappings | Nowhere | Not mapped |
| Components | Nowhere | Not built |

---

## Recommended Phase 1 Build Sequence

### Step 1: Doron runs Base44 agent prompt
Create MealPrepProfile, Recipe, RecipeIngredient entities with permissions (Authenticated Users Create/Read, Creator Only Update/Delete). Include subscription_tier and tier_trial_start on MealPrepProfile.

### Step 2: Hyphae builds config + onboarding
- Register meal_prep in workspaceTypes.js
- Add warm entry, workspace guide
- Build MealPrepOnboarding.jsx + page route

### Step 3: Hyphae builds Recipe Book tabs
MealPrepHome, RecipeBook, RecipeEditor, MealPrepSettings — all behind construction gate.

### Step 4: Hyphae updates server functions
- Add MealPrepProfile to getMyLaneProfiles
- Add meal-prep mappings to agentScopedWrite

### Step 5: Doron updates agentScopedQuery in Base44 dashboard

### Step 6: Hyphae builds MyLane card
RecipeBookCard component + myLaneRegistry.js entry.

### Step 7: Walkthrough with Doron → flip construction gate

---

## Surprises

1. **Repo path confusion:** The prompt referenced `locallane-spec-repo/` and `locallane-private/` but the actual paths are `Spec-Repo/` (capitals, public) and `private/` (lowercase, private). The `locallane-spec-repo` duplicate was deleted 2026-03-31.
2. **MEAL-PLANNING-CONCEPT.md exists** in the private repo — comprehensive vision doc from 2026-02-01 that aligns well with the workspace pattern.
3. **agentScopedQuery source is not in the repo** — lives only in Base44 dashboard. Cannot be updated by Hyphae.
4. **Warm entry message gaps** — Field Service and Property Management never got warm entry messages.
5. **Spec-Repo has more current docs** than community-node for STATUS-TRACKER and SEEDLING-TRACKER.

---

*Audit complete. The ground is mapped. Ready for planting when Doron runs the entity prompts.*

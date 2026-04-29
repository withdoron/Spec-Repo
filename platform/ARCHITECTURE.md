# Technical Architecture

> Detailed technical design for the LocalLane node ecosystem.

---

## Overview

LocalLane is a community platform built on Base44 with a React/Vite/Tailwind frontend. The architecture has three layers:

**Community Node** — The hub application (locallane.app). Directory, events, MyLane, business dashboard, admin panel. This is the production app serving real users.

**Standalone Nodes** — Independent Base44 apps (Contractor Daily, Property Pulse, Personal Finance) that serve individual users during the lab phase. They do NOT sync with the Community Node. Each proves its own value independently. See NODE-LAB-MODEL.md (private repo) for maturity criteria and integration path.

**Workspace Engine** — The emerging architecture (DEC-053) that generalizes the Dashboard from business-only to multi-type workspaces. A business is a workspace type. A team is a workspace type. Each fills the same shell (header, tabs, roles, Gold Standard) with different tools. See DASHBOARD-WORKSPACES-IMPLEMENTATION.md (private repo) for the full spec.

### Development Model

New features are built inside the Community Node with feature gating (DEC-049 Phase A):
- Admin-only visibility for unfinished features
- Tier-based access for premium features
- Role-based routing for different user types

Standalone nodes continue serving current users unchanged. They integrate into LocalLane only after proving maturity — real users, stable schema, tested roles (DEC-047).

### Future: Shared Backend (DEC-049 Phase B)

When ready to scale beyond a single app, Base44's Backend Platform supports shared auth and entities across multiple frontends deployed as subdomains (contractor.locallane.app, property.locallane.app). This is planned but not active.

---

## Node Architecture

### Current State

```
Community Node (locallane.app) — the hub, production app
├── Directory, Events, MyLane, Business Dashboard, Admin Panel
├── Workspace Engine (DEC-053) — foundation queued
│   ├── Business workspace type (existing, to be refactored)
│   └── Team workspace type (Play Trainer — DEC-054, queued)
└── All new vertical features built here with feature gating

Standalone Nodes (lab phase — independent, no sync)
├── Contractor Daily — Build 17, field testing with Dan Sikes
├── Property Pulse — Phase 2 complete, family users
├── Personal Finance — Phase 1 complete, Doron
└── Field Service Engine — cloned from Contractor Daily, active development
```

### Integration Path

Standalone nodes do not sync today. When a node reaches maturity (NODE-LAB-MODEL.md), it integrates via one of two paths:

1. **Rebuild inside Community Node** (Phase A — current approach) — Feature gating, workspace types, role-based routing
2. **Shared Base44 Backend** (Phase B — future) — Multiple frontends, shared auth/entities, custom subdomains

### Archived: Sync Patterns (Future Reference)

The original architecture described parent-child sync patterns (config push, data push, health checks). These become relevant during Phase B integration or if nodes need real-time data exchange. For now, each node is fully independent.

---

## Data Models

### Entity Definitions

Entity definitions live in the Base44 dashboard, not in code. There is no `base44/entities/` folder in the repo. Entities are accessed via `base44.entities.<EntityName>`.

For the full list of entities and their security status, see claude.md in the community-node repo and DEC-025 in DECISIONS.md.

### Key Entity Relationships

```
User
├── owns → Business (via owner_user_id)
├── has → JoyCoins (balance, transactions)
├── has → RSVP (event attendance)
├── has → Recommendation (nods, stories, vouches)
├── has → NewsletterSubscriber
└── future: TeamMember (via workspace engine)

Business
├── hosts → Event
├── has → AccessWindow (Joy Coins hours)
├── has → Location
├── has → Recommendation (received)
└── config → AdminSettings (staff_roles, platform_config)

Event
├── has → RSVP
├── belongs to → Business
├── tagged with → Networks, Event Types
└── has → Joy Coins settings (cost, capacity, frequency limits)
```

### Joy Coins (Community Pass)

Joy Coins are the token system within the Community Pass membership. Key entities:

- **JoyCoins** — Per-user balance tracking
- **JoyCoinTransactions** — Immutable audit trail (redemptions, allocations, expirations)
- **JoyCoinReservations** — Holds placed during RSVP, redeemed on check-in

Joy Coins are non-transferable between members (DEC-041). Unused coins expire monthly and flow to the Community Scholarship Pool.

### Config System

Platform configuration uses the AdminSettings entity as a key-value store (DEC-016):

- `staff_roles:{business_id}` — Staff role assignments
- `staff_invites:{business_id}` — Pending staff invites
- `platform_config:{domain}:{config_type}` — Platform-wide configuration (networks, event types, age groups, durations)

---

## Security Model

### Authentication

Base44 handles authentication. Users log in via Base44's built-in auth system.

### Entity Permissions (DEC-025)

Entity permissions are locked down in Base44's dashboard. Three phases:

- **Phase 1 & 2 (Complete):** AdminAuditLog, Concern, Region, Archetype, CategoryGroup, SubCategory locked. *(PunchPass entities deleted from Base44 per DEC-028.)*
- **Phase 3 (Deferred):** Business, Event, AdminSettings, RSVP, Location, User, Recommendation, Spoke, SpokeEvent, CategoryClick require service role function migration before permissions can be tightened.

### Server Functions

Sensitive writes use Base44 server functions with `base44.asServiceRole` for elevated permissions. The server function verifies the calling user's role before executing:

- `updateBusiness` — Owner/co-owner/admin auth, field allowlist, slug collision handling
- `updateUser` — Onboarding fields only
- `setJoyCoinPin` / `validateJoyCoins` — Joy Coins PIN operations
- `handleEventCancellation` — Refund/forfeit logic

### Security Hardening (Post-Launch)

Business entity Update is currently "No restrictions" to unblock admin during pilot. Needs server function that checks platform admin role before allowing updates to businesses the admin didn't create.

---

## Observability

Logging and metrics are handled through browser console during pilot. Structured observability (error tracking, usage metrics) is a post-launch concern. Console.log cleanup was completed 2026-02-20.

---

## Deployment

### Current Flow

```
Cursor (edit) → GitHub (push) → Base44 (auto-sync) → Manual Publish → Live
```

Base44 auto-syncs from GitHub. Preview in Base44's editor before publishing to production. Rollback by reverting the commit in GitHub and re-syncing.

No staging environment exists. Base44 has this on their roadmap.

---

## Community Health Data Layer

LocalLane's north star is making community health visible (DEC-029). This requires a data pipeline that transforms platform activity into vitality signals.

### Data Signals

The platform already generates the raw signals needed:

| Signal | Source Entity | Already Queried? |
|--------|--------------|-----------------|
| RSVPs created | RSVP | ✅ Yes (MyLane) |
| Event attendance (check-ins) | JoyCoinTransactions | ✅ Yes (CheckIn) |
| Recommendations given | Recommendation | ✅ Yes (MyLane) |
| Community Pass activity | JoyCoins | ✅ Yes (MyLane) |
| Days active | User (last_active) | ⚠️ Needs field |
| New businesses | Business (created_date) | ✅ Yes (MyLane) |
| Concerns resolved | Concern (status) | ✅ Yes (Admin) |

### Vitality Computation

Vitality scores are calculated client-side from data already fetched. No new API calls needed for Phase 1. The score drives visual state but is never shown as a number.

Three scales (fractal pattern):
- **Personal** — one user's community participation
- **Network** — aggregate across a pass network's members
- **Community** — aggregate of all networks plus community-wide signals

Vitality formulas and thresholds are documented in the private repo (ORGANISM-CONCEPT.md).

### Organism Component Architecture

```
useVitality(userId)          → Computes personal vitality from existing data
CommunityOrganism.jsx       → Renders animated visual based on vitality state
  ├── Phase 1: CSS + SVG     (no dependencies)
  ├── Phase 2: Lottie         (mood responses)
  ├── Phase 3: Canvas         (network identity blending)
  └── Phase 4: Three.js       (social organisms)
```

The organism is ambient, not dominant. It lives in MyLane GreetingHeader and reflects real data — not artificial engagement signals.

---

## Backup & Recovery

### What's Backed Up

- **Base44:** Handles database backups automatically
- **GitHub:** Full code history with every commit
- **Spec-Repo:** Decision history, architecture docs

### Recovery Scenarios

| Scenario | Recovery |
|----------|----------|
| Bad code deployed | Revert commit in GitHub, re-sync to Base44 |
| Data corrupted | Contact Base44 support for backup restore |
| Config mistake | Config is versioned; roll back to previous version |
| Node completely broken | Worst case: re-deploy from GitHub |

---

*This document is maintained in the Spec-Repo. Update when architecture changes.*

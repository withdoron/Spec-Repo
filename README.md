# LocalLane Ecosystem — Master Specification

> **Source of Truth** for the LocalLane node-based platform.  
> Last updated: 2026-01-27

---

## Table of Contents

1. [Vision & Problem Statement](#vision--problem-statement)
2. [Goals and Non-Goals](#goals-and-non-goals)
3. [System Architecture](#system-architecture)
4. [The Node Model](#the-node-model)
5. [Archetypes](#archetypes)
6. [Tier System](#tier-system)
7. [Trust & Reputation](#trust--reputation)
8. [Punch Pass System](#punch-pass-system)
9. [User Workflows](#user-workflows)
10. [Data Flow & Dynamic Configuration](#data-flow--dynamic-configuration)
11. [API Contracts](#api-contracts)
12. [Technical Stack](#technical-stack)
13. [Rollout Plan](#rollout-plan)
14. [Open Questions](#open-questions)
15. [Decision Log](#decision-log)

---

## Vision & Problem Statement

### The Problem

Local communities lack a unified, trustworthy platform to discover businesses, events, and services. Existing solutions (Yelp, Facebook Events, Google Maps) are:

- **Ad-driven** — paid placement beats quality
- **Fragmented** — events in one app, businesses in another, community groups in a third
- **Not community-owned** — no local accountability or trust

### The Vision

**LocalLane** is a trust-based ecosystem where:

- **Reputation is earned, not bought** — user ratings drive visibility
- **Businesses can graduate** — from free listing → paid features → their own Partner Node (mini-app)
- **Data flows intelligently** — local events appear in the community hub automatically
- **The community scales** — Community Nodes → Regional Nodes → National (future)

### The Tagline

> "One place to find what you need in your local community — powered by trust, not ads."

---

## Goals and Non-Goals

### Goals (Next 90 Days)

| # | Goal | Success Metric |
|---|------|----------------|
| 1 | Launch pilot with Event Node + Community Node | 10+ real events posted by homeschool group |
| 2 | Punch pass system working (demo mode) | Users can "purchase" and "redeem" test passes |
| 3 | Prove the node model works | Event Node creates event → appears in Community Node automatically |
| 4 | Onboard first test archetype | Church/nonprofit node functional |
| 5 | Document everything | Spec-Repo is complete and Cursor can follow it |

### Non-Goals (Explicitly Out of Scope)

- Real payment processing (Phase 2+)
- Regional Node implementation (future)
- Mobile native apps (web-first)
- Multi-tenancy / white-labeling
- ML-based recommendations
- Complex fraud detection (simple trust metrics first)

---

## System Architecture

### Node Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    [FUTURE: National Node]                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    [FUTURE: Regional Node]                  │
│                    (covers multiple communities)            │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                     COMMUNITY NODE                          │
│                     (LocalLane Hub)                         │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ • Directory of all local businesses                    │ │
│  │ • Aggregated events from all Event Nodes               │ │
│  │ • Punch pass purchase & management                     │ │
│  │ • User accounts & profiles                             │ │
│  │ • Trust/reputation display                             │ │
│  │ • Network configuration (TCA, Recess, Homeschool, etc) │ │
│  └────────────────────────────────────────────────────────┘ │
└──────────┬─────────────────────┬─────────────────┬──────────┘
           │                     │                 │
     ┌─────▼─────┐        ┌──────▼──────┐   ┌─────▼─────┐
     │  EVENT    │        │  NONPROFIT  │   │  PARTNER  │
     │  NODE     │        │  NODE       │   │  NODE     │
     │ (Tier 3)  │        │  (future)   │   │ (Tier 3)  │
     └───────────┘        └─────────────┘   └───────────┘
```

### Data Flow Directions

| Direction | What Flows | Example |
|-----------|-----------|---------|
| **Down** (Parent → Child) | Configuration, categories, network lists, shared settings | Community Node adds "Soccer League" to networks → all Partner Nodes see it |
| **Up** (Child → Parent) | Events, listings, transactions, ratings | Event Node creates "Movie Night" → appears in Community Node calendar |
| **Sideways** (Node ↔ Node) | Punch pass redemptions, cross-promotions | User buys pass on Community Node → redeems at Jiu-Jitsu Partner Node |

---

## The Node Model

### What Is a Node?

A **node** is a self-contained application that:

1. **Serves a specific purpose** (events, directory, nonprofit management)
2. **Operates independently** (can function if other nodes are down)
3. **Connects via API** (shares data with parent/child nodes)
4. **Has its own data store** (but pulls configuration from parent)

### Node Responsibilities

| Aspect | Node's Job | Parent's Job |
|--------|-----------|--------------|
| **UI/UX** | Full control — owns its screens, flows, branding | Provides style guide & shared components |
| **Business Logic** | Owns its features (event creation, booking, etc.) | None — doesn't dictate child behavior |
| **Data Storage** | Stores its own data (events, users, transactions) | Provides config (categories, networks, settings) |
| **Authentication** | Accepts auth tokens from parent OR manages its own | Issues tokens, manages user accounts |
| **Sync** | Pushes relevant data up (events, ratings) | Aggregates and displays child data |

### Node Anatomy (File Structure)

Every node follows this structure:

```
node-name/
├── src/
│   ├── components/      # Reusable UI pieces
│   ├── pages/           # Screen-level components
│   ├── api/             # API calls to parent/children
│   ├── config/          # Dynamic config (pulled from parent)
│   ├── hooks/           # Shared React hooks
│   └── utils/           # Helper functions
├── functions/           # Backend/serverless functions
├── tailwind.config.js   # Styling (must match ecosystem palette)
├── package.json
└── README.md            # Node-specific documentation
```

### How Nodes Communicate

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **REST API** | Real-time data needs, user actions | User RSVPs to event → Event Node calls Community Node API |
| **Webhook** | When child needs to notify parent of changes | Event created → Event Node sends webhook to Community Node |
| **Polling** | Config sync, non-urgent updates | Partner Node checks Community Node every 5 min for new networks |

### Versioning Strategy

- **API versions:** `/api/v1/events`, `/api/v2/events`
- **Breaking changes:** New version, old version supported for 30 days
- **Config changes:** Always backwards-compatible (add fields, don't remove)

### Preventing Tight Coupling

| Risk | Mitigation |
|------|-----------|
| Parent changes break children | Config is versioned; children specify which version they support |
| Child depends on parent being online | Children cache config locally; graceful degradation |
| Shared code creates dependencies | Shared code lives in `@locallane/common` package (future); for now, copy-paste is OK |

---

## Archetypes

An **archetype** is a template for a type of business or organization. Each archetype defines:

- What fields/data this type needs
- What features are available at each tier
- What UI patterns make sense

### Current Archetypes

| Archetype | Description | Status |
|-----------|-------------|--------|
| **Event Coordinator** | Posts events, manages RSVPs, accepts punch passes | ✅ In Progress |
| **Nonprofit/Church** | Member directory, announcements, volunteer coordination | 🔜 Next |
| **Generic Business** | Basic listing in community directory | 🔜 After Nonprofit |

### Future Archetypes (Backlog)

- Mechanic Shop
- Online Store
- Jiu-Jitsu / Martial Arts
- Homeschool Co-op
- Restaurant / Food Vendor
- Service Provider (plumber, electrician, etc.)

### Archetype Spec Template

See `/ARCHETYPES/_template.md` for the blueprint to create new archetypes.

---

## Tier System

Every archetype can exist at three tiers:

### Tier Overview

| Tier | Name | Cost | Where They Live | Key Features |
|------|------|------|-----------------|--------------|
| **1** | Free | $0 | Community Node | Basic listing, shows in directory |
| **2** | Pro | TBD | Community Node | Enhanced features, punch pass acceptance, analytics |
| **3** | Partner | Earned | Own Node | Full mini-app, maximum customization, premium trust badge |

### How Tier 3 is Earned (Not Bought)

Tier 3 (Partner Node) is a privilege, not a purchase. Criteria (to be refined):

| Metric | Threshold | Why It Matters |
|--------|-----------|----------------|
| User rating | 4.5+ stars average | Proves quality |
| Time active | 6+ months at Tier 2 | Proves consistency |
| Engagement | X events/month or Y transactions | Proves activity |
| Community vouches | 3+ other Tier 2/3 partners vouch | Proves reputation |
| No violations | Zero trust violations | Proves integrity |

**Open Question:** Should criteria be archetype-specific or universal? → See Decision Log.

### Feature Unlock by Tier

Features are visible but grayed out at lower tiers. Upgrading unlocks them.

| Feature | Tier 1 | Tier 2 | Tier 3 |
|---------|--------|--------|--------|
| Basic listing in directory | ✅ | ✅ | ✅ |
| Post events | ✅ | ✅ | ✅ |
| Accept punch passes | ❌ | ✅ | ✅ |
| Analytics dashboard | ❌ | ✅ | ✅ |
| Custom branding | ❌ | ❌ | ✅ |
| Own Partner Node | ❌ | ❌ | ✅ |
| Priority in search results | ❌ | ❌ | ✅ |
| Trust badge | ❌ | ❌ | ✅ |

---

## Trust & Reputation

### Core Principle

> Visibility is earned through trust, not payment.

### How Trust is Calculated

| Signal | Weight | Description |
|--------|--------|-------------|
| User ratings | 40% | Average star rating from community members |
| Rating volume | 20% | More ratings = more confidence in the score |
| Activity | 15% | Regular posting, event hosting, engagement |
| Tenure | 10% | How long they've been active |
| Vouches | 10% | Endorsements from other trusted businesses |
| Violations | -X% | Deductions for reported issues |

### Trust Display

- **New businesses:** "New to LocalLane" badge (neutral, not penalized)
- **Building trust:** 1-3 stars shown, "Building reputation" label
- **Trusted:** 4+ stars, full display
- **Partner:** Special badge, priority placement

### Preventing Fake Reviews

| Risk | Mitigation |
|------|-----------|
| Fake accounts leaving reviews | Users must have activity history to rate |
| Business rating themselves | Can't rate businesses you own/manage |
| Review bombing | Rate limiting, anomaly detection (future) |
| Paid reviews | Community reporting, pattern detection (future) |

**Phase 1:** Simple protections (rate limiting, ownership checks)  
**Phase 2:** More sophisticated detection

---

## Punch Pass System

### What It Is

A **punch pass** is a prepaid card that works across participating businesses. Think of it like a local gift card network.

### How It Works

```
┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│   USER           │         │  COMMUNITY NODE  │         │  PARTNER NODE    │
│                  │         │                  │         │  (Jiu-Jitsu)     │
│  "Buy Recess     │────────▶│  Creates pass    │         │                  │
│   Punch Pass"    │         │  with 10 punches │         │                  │
│                  │         │  Balance: $50    │         │                  │
│                  │         │                  │         │                  │
│  "Use pass at    │         │                  │         │                  │
│   Jiu-Jitsu"     │─────────┼─────────────────▶│  Verify pass     │
│                  │         │                  │◀────────│  Deduct punch    │
│                  │         │  Update balance  │         │  Confirm         │
└──────────────────┘         └──────────────────┘         └──────────────────┘
```

### Pass Types (Networks)

| Network | Example Businesses | Target Audience |
|---------|-------------------|-----------------|
| Recess | Bowling, Jiu-Jitsu, Trampoline Park | Families, homeschoolers |
| TCA | TCA member businesses | TCA community |
| Homeschool | Educational vendors, co-ops | Homeschool families |

### Demo Mode (Phase 1)

- Passes are "purchased" with fake money
- Full UX as if real (prices shown, receipts generated)
- No actual payment processing
- Data tracked for future real implementation

### Real Mode (Phase 2+)

- Integrate Stripe or similar
- Actual payment processing
- Refund policy required
- Terms of Service required

---

## User Workflows

### Workflow 1: Event Discovery (End User)

```
1. User opens Community Node
2. Browses "Events" section
3. Sees events from all connected Event Nodes
4. Filters by date, category, network (Recess, Homeschool, etc.)
5. Clicks event → sees details
6. RSVPs (if logged in) or bookmarks (if guest)
```

### Workflow 2: Event Creation (Event Coordinator - Tier 2+)

```
1. Event Coordinator logs into their Event Node
2. Clicks "Create Event"
3. Fills in: title, date, location, description
4. Selects networks (Recess, Homeschool, etc.) — list pulled from Community Node
5. Sets punch pass acceptance (if Tier 2+)
6. Publishes
7. Event automatically appears in Community Node
```

### Workflow 3: Punch Pass Purchase (End User)

```
1. User opens Community Node
2. Goes to "Punch Passes"
3. Selects "Recess Pass - 10 punches - $50" (demo mode)
4. "Purchases" (no real money in Phase 1)
5. Pass appears in their wallet
6. Can view balance, transaction history
```

### Workflow 4: Punch Pass Redemption (End User + Business)

```
1. User arrives at Jiu-Jitsu Partner Node business
2. Opens their wallet, shows pass QR code
3. Business scans code (or enters code manually)
4. System verifies pass, deducts punch
5. Both parties see confirmation
```

---

## Data Flow & Dynamic Configuration

### The Problem We're Solving

If we hardcode configuration (like network lists) into each node:

- Adding a new network = updating every node manually
- Inconsistency risk = Node A has "Soccer League", Node B doesn't
- Maintenance nightmare = doesn't scale

### The Solution: Config Flows Down

```
Community Node (Source of Truth)
├── networks: ["Recess", "TCA", "Homeschool", "Soccer League"]
├── categories: ["Sports", "Education", "Entertainment", "Food"]
├── punchPassTypes: [{name: "Recess", price: 50, punches: 10}, ...]
│
└── Partner/Event Nodes (Consumers)
    └── On startup: fetch config from parent
    └── Cache locally (works offline)
    └── Re-fetch periodically (every 5 min or on-demand)
```

### Config Schema (v1)

```json
{
  "version": "1.0",
  "lastUpdated": "2026-01-27T12:00:00Z",
  "networks": [
    {"id": "recess", "name": "Recess", "active": true},
    {"id": "tca", "name": "TCA", "active": true},
    {"id": "homeschool", "name": "Homeschool Network", "active": true}
  ],
  "categories": [
    {"id": "sports", "name": "Sports & Fitness"},
    {"id": "education", "name": "Education"},
    {"id": "entertainment", "name": "Entertainment"}
  ],
  "punchPasses": [
    {"id": "recess-10", "network": "recess", "name": "Recess 10-Pack", "price": 50, "punches": 10}
  ]
}
```

### Config Update Flow

1. Admin updates config in Community Node admin panel
2. Config is versioned (v1.0 → v1.1)
3. Child nodes detect new version on next poll
4. Child nodes fetch and cache new config
5. Old config kept for 30 days (rollback safety)

---

## API Contracts

### Event Node → Community Node

#### POST /api/v1/events

Creates or updates an event in the community hub.

```json
// Request
{
  "sourceNodeId": "events-node-001",
  "event": {
    "id": "evt-123",
    "title": "Family Movie Night",
    "date": "2026-02-14T18:00:00Z",
    "location": "Community Center",
    "description": "...",
    "networks": ["recess", "homeschool"],
    "acceptsPunchPass": true,
    "punchPassTypes": ["recess-10"]
  }
}

// Response
{
  "success": true,
  "communityEventId": "comm-evt-456"
}
```

#### DELETE /api/v1/events/:id

Removes an event from the community hub.

### Community Node → Child Nodes

#### GET /api/v1/config

Returns current configuration for child nodes.

```json
// Response
{
  "version": "1.0",
  "config": { /* see Config Schema above */ }
}
```

#### POST /api/v1/punch-pass/verify

Verifies and redeems a punch pass.

```json
// Request
{
  "passId": "pass-789",
  "businessId": "biz-456",
  "punchesRequested": 1
}

// Response
{
  "valid": true,
  "remainingPunches": 7,
  "transactionId": "txn-abc"
}
```

---

## Technical Stack

### Current Stack (Base44)

| Layer | Technology | Notes |
|-------|------------|-------|
| **Frontend** | React + Vite | Fast builds, modern React |
| **Styling** | Tailwind CSS | Utility-first, easy theming |
| **Backend** | Base44 Functions | Serverless, managed by Base44 |
| **Database** | Base44 built-in | Document store |
| **Hosting** | Base44 | Auto-deploy on publish |
| **Auth** | Base44 built-in | User management included |

### Development Tools

| Tool | Purpose |
|------|---------|
| **Cursor** | AI-assisted code editor |
| **GitHub** | Source control, version history |
| **Claude** | Principal engineer / architect (me) |

### Shared Dependencies

All nodes should use consistent versions:

```json
{
  "react": "^18.x",
  "tailwindcss": "^3.x",
  "vite": "^5.x"
}
```

---

## Rollout Plan

### Phase 0: Foundation (Current Week)

- [x] Create Spec-Repo structure
- [ ] Confirm Cursor → GitHub → Base44 flow works
- [ ] Extract and document style guide
- [ ] Review Community Node, decide salvage vs rebuild
- [ ] Clean up / stabilize Event Node

### Phase 1: Pilot Launch (Week 2)

- [ ] Event Node fully functional
- [ ] Community Node displays events
- [ ] Punch pass system (demo mode)
- [ ] Basic user accounts
- [ ] Soft launch with homeschool group

### Phase 2: Nonprofit Archetype (Weeks 3-4)

- [ ] Create nonprofit/church node
- [ ] Member directory
- [ ] Announcements
- [ ] Volunteer coordination
- [ ] Test with your church

### Phase 3: Trust & Polish (Month 2)

- [ ] Implement trust/rating system
- [ ] Define Tier 3 earned criteria
- [ ] Onboard more businesses
- [ ] Gather feedback, iterate

### Phase 4: Regional Scale (Month 3+)

- [ ] Design Regional Node
- [ ] Multi-community support
- [ ] Real payment processing
- [ ] Terms of Service / legal

---

## Open Questions

| # | Question | Status | Owner |
|---|----------|--------|-------|
| 1 | Should Tier 3 criteria be archetype-specific or universal? | Open | Product decision |
| 2 | What's the pricing for Tier 2? | Open | Business decision |
| 3 | How do we handle business closures with active punch passes? | Open | Legal/policy |
| 4 | What happens if a Partner Node goes offline? | Open | Technical design |
| 5 | Do users need accounts to browse, or only to interact? | Open | UX decision |

---

## Decision Log

| Date | Decision | Reasoning | Made By |
|------|----------|-----------|---------|
| 2026-01-27 | Repos will be public for now | Faster iteration, code theft risk low at this stage, can go private later | Doron + Claude |
| 2026-01-27 | Punch pass starts in demo mode | Avoids payment processing complexity for pilot | Doron + Claude |
| 2026-01-27 | Spec-Repo is source of truth | Solves AI memory problem, keeps all decisions documented | Doron + Claude |
| 2026-01-27 | Tier 3 is earned, not purchased | Core differentiator — trust over money | Doron |
| 2026-01-27 | Next archetype is nonprofit/church | Real test case available (Doron's church), similar to events | Doron + Claude |

---

## Related Documents

- [ARCHITECTURE.md](./ARCHITECTURE.md) — Detailed technical architecture
- [STYLE-GUIDE.md](./STYLE-GUIDE.md) — Colors, fonts, UI patterns
- [DECISIONS.md](./DECISIONS.md) — Full decision log with context
- [ARCHETYPES/_template.md](./ARCHETYPES/_template.md) — Blueprint for new archetypes
- [ARCHETYPES/event-node.md](./ARCHETYPES/event-node.md) — Event archetype spec
- [CHECKLISTS/](./CHECKLISTS/) — Step-by-step procedures

---

*This document is maintained in the Spec-Repo and should be updated whenever major decisions are made or the system evolves.*

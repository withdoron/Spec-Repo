# LocalLane Ecosystem Specification

> Source of truth for the LocalLane platform architecture, design standards, and development workflows.

---

## Quick Links

| Document | Purpose |
|----------|---------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Technical patterns, data models, APIs |
| [STYLE-GUIDE.md](./STYLE-GUIDE.md) | Visual design standards (Gold Standard) |
| [COMPONENTS.md](./COMPONENTS.md) | Detailed UI component specifications |
| [TIER-SYSTEM.md](./TIER-SYSTEM.md) | Tier definitions and feature matrix |
| [ENTITY-SYSTEM.md](./ENTITY-SYSTEM.md) | Archetype and goals-based dashboard |
| [CURRENT-STATE.md](./CURRENT-STATE.md) | What currently exists |
| [INTEGRATION-PLAN.md](./INTEGRATION-PLAN.md) | Phase 2 implementation steps |
| [DECISIONS.md](./DECISIONS.md) | Decision log with context |
| [WORKFLOW.md](./WORKFLOW.md) | Development workflow |

---

## What is LocalLane?

**Mission:** Revitalize local commerce by connecting communities with local experiences, while actively funding the next generation through youth scholarships and community programming.

**Vision:** A "Main Street" digital infrastructure where businesses, organizations, and residents connect through trusted, steward-curated listings and events—scalable city-by-city.

**Positioning:**
- Neutral local utility (not an ideological platform)
- Trust-first: real human stewards, verification, transparent labels
- Human curation over algorithms
- Local resilience: highlight barter/silver and local sourcing

---

## Core Concepts

### The Cell + Steward Model

**Cell** = A local service area (city/town/county)
- Prototype: Greater Eugene-Springfield
- Scaling unit for expansion

**Area Steward** = Human curator per Cell
- Onboards and verifies entities
- Reviews and moderates submissions
- Curates local "vibe"
- Runs local newsletter/comms
- Earns commission on subscriptions and transactions

### Node Architecture

```
Regional Node (future)
  └── Community Node (the hub - locallane.app)
      └── Partner Nodes (Tier 3 businesses with own apps)
          └── Example: Event Node, Nonprofit Node, etc.
```

**Data Flow:**
- Config flows DOWN (parent → child)
- Content flows UP (child → parent)
- Punch passes work ACROSS nodes

### Entity Types

LocalLane supports many entity types, not just businesses:
- Businesses / services (commercial)
- Nonprofits
- Churches / faith communities
- Community groups / clubs
- Farms / CSAs / market vendors
- Schools / co-ops / tutors
- Venues
- Microbusinesses / homestands
- Individuals (for micro activity)

---

## Tier System

See [TIER-SYSTEM.md](./TIER-SYSTEM.md) for complete documentation.

### Quick Reference

| Tier Level | Code Value | Display Name | Cost |
|------------|------------|--------------|------|
| Tier 1 | `basic` | Basic | Free |
| Tier 2 | `standard` | Standard | $X/month |
| Tier 3 | `partner` | Partner | Earned + $Y/month |

**Always use code values** (`basic`, `standard`, `partner`) in database and code.

### Tier 3 Criteria (Earned)
- 4.5+ star rating
- 6+ months active at Tier 2
- Engagement threshold met
- 3+ vouches from other partners
- Zero policy violations

### Network Membership Add-On
- **Cost:** $18/month
- **Benefit:** Accept Punch Passes, appear in network filters
- **Included in:** Partner tier

---

## Network Brands

| Network | Focus | Target |
|---------|-------|--------|
| **Recess** | Movement, fitness, play | Kids & families |
| **TCA** (The Creative Alliance) | Arts, trades, skills, enrichment | All ages |
| **Homeschool** | Educational activities | Homeschool families |

---

## Punch Pass System

### Consumer Packs

| Pack | Price | Per Ticket | Savings |
|------|-------|------------|---------|
| 10 tickets | $100 | $10.00 | Base |
| 20 tickets | $180 | $9.00 | ~10% |
| 30 tickets | $255 | $8.50 | ~15% |

**Hero product:** 20-pack

### How It Works
1. User buys a Punch Pack (tickets, not dollars)
2. Network businesses price in tickets (1-3 per activity)
3. User pays with tickets at events
4. Business receives payout minus 15% platform fee
5. Packs expire 12 months from purchase (FIFO)

### Phase 1: Demo Mode
- Full UX without real payment processing
- Shows prices, tracks balances
- Clear "demo mode" indication
- Real payments in Phase 2

---

## The "Gold Standard" Design System

From the Blueprint:
> "Local Lane is a premium, trusted community platform. We avoid generic 'template' colors."

### Color Palette

```
BACKGROUNDS:
  Page:        bg-slate-950   #020617
  Surface:     bg-slate-900   #0f172a
  Elevated:    bg-slate-800   #1e293b

ACCENT (Gold):
  Primary:     bg-amber-500   #f59e0b
  Hover:       bg-amber-400   #fbbf24
  Active:      bg-amber-600   #d97706

TEXT:
  Primary:     text-white / text-slate-100
  Secondary:   text-slate-300
  Muted:       text-slate-400
  On Gold:     text-black
```

### Rules
- Dark theme only (no light backgrounds)
- Icons: Gold or White only
- No functional color-coding (no red hearts, green checkmarks)
- See [STYLE-GUIDE.md](./STYLE-GUIDE.md) for complete details

---

## Current Implementation Status

### Event Node (Partner Node Template)
- ✅ Event creation/edit/delete
- ✅ Sync to Community Node
- ✅ Network tagging (Recess, etc.)
- ✅ Punch pass integration (UI)
- ✅ Dashboard with stats
- ✅ Punch Pass earnings display
- ⏳ Organizer name display (shows "Event Spoke" instead of business name)

### Community Node
- ✅ Browse Events view
- ✅ Event detail modal
- ✅ Quick filters (All, This Weekend, Today)
- ✅ Punch Pass badge display
- ✅ Network badge display
- ⚠️ Admin Panel (light theme - needs dark)
- ⏳ User registration/accounts
- ⏳ Punch Pass purchase flow
- ⏳ RSVP functionality
- ⏳ Business directory (deferred to post-pilot)

---

## Revised 7-Day Pilot Plan

**Goal:** Working event flow for homeschool pilot

### Day 1 (Today) ✅
- [x] Spec-Repo populated and pushed
- [x] Style guide complete
- [x] Development workflow documented
- [x] Current state assessed

### Day 2: Community Node Cleanup
- [ ] Fix organizer name display (show business name, not "Event Spoke")
- [ ] Make Admin Panel dark theme
- [ ] Remove/hide unused features (directory, boost, etc.)
- [ ] Verify event sync is stable

### Day 3: User Flow
- [ ] User registration/login flow
- [ ] RSVP to events (basic)
- [ ] "My Saved Events" view

### Day 4: Punch Pass Demo Mode
- [ ] Purchase flow (demo - no real payment)
- [ ] User wallet/balance display
- [ ] Redeem flow at RSVP

### Day 5: Event Node Polish
- [ ] Fix any sync issues
- [ ] Verify business name flows through
- [ ] Test create → sync → display flow

### Day 6: End-to-End Testing
- [ ] Full flow: Create event → Appears in Community → User RSVPs
- [ ] Punch pass flow: Purchase → Redeem
- [ ] Mobile testing
- [ ] Fix critical bugs

### Day 7: Pilot Prep
- [ ] Add 5-10 real test events
- [ ] Brief pilot users (homeschool group)
- [ ] Set up feedback collection
- [ ] Soft launch

---

## Repository Structure

```
Spec-Repo/
├── README.md           # This file
├── ARCHITECTURE.md     # Technical patterns
├── STYLE-GUIDE.md      # Visual standards
├── COMPONENTS.md       # UI component specs
├── DECISIONS.md        # Decision log
├── CURSOR-RULES.md     # Rules for Cursor agent
├── WORKFLOW.md         # Development workflow
├── archetypes/
│   ├── _template.md    # Template for new archetypes
│   ├── event-node.md   # Event coordinator archetype
│   └── nonprofit-node.md # Church/nonprofit archetype
└── checklists/
    ├── 7-day-plan.md   # Execution plan
    ├── new-node.md     # New node setup
    └── pilot-launch.md # Pre-launch checklist
```

---

## Related Repositories

| Repo | Purpose | URL |
|------|---------|-----|
| **community-node** | The main hub app | github.com/withdoron/community-node |
| **events-node** | Partner node for event organizers | github.com/withdoron/events-node |
| **Spec-Repo** | This specification repo | github.com/withdoron/Spec-Repo |

---

## Tech Stack

Both nodes use identical setup:
- **React + Vite** - Frontend framework
- **Tailwind CSS** - Styling
- **shadcn/ui** - Component library
- **@tanstack/react-query** - Data management
- **Base44** - Hosting, backend, auth
- **Lucide React** - Icons

---

## Development Workflow

```
Claude (Strategy) → Spec-Repo → Cursor (Code) → GitHub → Base44 → Live
```

1. **Claude** reviews/updates Spec-Repo (architecture, decisions)
2. **Cursor** reads Spec-Repo, implements code
3. Code pushed to **GitHub**
4. **Base44** auto-syncs from GitHub
5. Manual **publish** makes changes live

See [WORKFLOW.md](./WORKFLOW.md) for details.

---

## Starting a New Conversation

When starting a new Claude conversation:

```
I'm working on the LocalLane ecosystem. Please review:

Spec-Repo (Google Docs mirror):
- [Link to README]
- [Link to ARCHITECTURE]
- [Link to STYLE-GUIDE]

Key context:
1. Node-based architecture (Community Node + Partner Nodes)
2. Tier system (Standard → Storefront → Partner)
3. "Gold Standard" dark theme with amber/gold accent
4. Currently preparing for homeschool pilot

Today I'm working on: [YOUR TASK]
```

---

## Open Questions

1. Tier 3 criteria: archetype-specific or universal?
2. Tier 2 pricing: $X/month - what's X?
3. Business closures with active punch passes - refund policy?
4. Family accounts (parent + kids) - how to handle?
5. Partner profit sharing model (for later)

---

## Contact

- **Owner:** Doron
- **Technical Lead:** Claude + Cursor
- **Email:** doron.bsg@gmail.com

---

*This spec is the external brain for LocalLane development. Always reference before implementing.*

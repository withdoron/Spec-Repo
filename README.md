# LocalLane Ecosystem Specification

> Source of truth for the LocalLane platform architecture, design standards, and development workflows.

---

## Quick Links

| Document | Purpose |
|----------|---------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Technical patterns, data models, vitality data layer |
| [STYLE-GUIDE.md](./STYLE-GUIDE.md) | Visual design standards (Gold Standard) |
| [TIER-SYSTEM.md](./TIER-SYSTEM.md) | Tier definitions and feature matrix |
| [RECOMMENDATION-SYSTEM.md](./RECOMMENDATION-SYSTEM.md) | Trust-first Nod/Story/Vouch system |
| [ENTITY-SYSTEM.md](./ENTITY-SYSTEM.md) | Archetype and goals-based dashboard |
| [MYLANE.md](./MYLANE.md) | MyLane spec — user dashboard with organism placement |
| [USER-TIERS.md](./USER-TIERS.md) | User tier definitions (Free/Member) |
| [STATUS-TRACKER.md](./STATUS-TRACKER.md) | Project status, roadmap, and session logs |
| [STRIPE-CONNECT.md](./STRIPE-CONNECT.md) | Stripe integration (Community Pass model) |
| [BUILD-PROTOCOL.md](./BUILD-PROTOCOL.md) | Universal build sequence — how every feature ships. ✅ Current |
| [BUSINESS-DASHBOARD-v2.md](./BUSINESS-DASHBOARD-v2.md) | Business Dashboard v2 strategy spec (replaces INTEGRATION-PLAN.md) |
| [ADMIN-ARCHITECTURE.md](./ADMIN-ARCHITECTURE.md) | Admin panel structure |
| [DECISIONS.md](./DECISIONS.md) | Decision log with context |
| [WORKFLOW.md](./WORKFLOW.md) | Development workflow and session SOP |
| [BOOK-FRAMEWORKS.md](./BOOK-FRAMEWORKS.md) | Marketing and business frameworks research |

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
- Joy Coins work within Community Pass network

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
| Tier 2 | `standard` | Standard | $79/month |
| Tier 3 | `partner` | Partner | $149/month (earned) |

**Always use code values** (`basic`, `standard`, `partner`) in database and code.

### Tier 3 Criteria (Earned)
- 4.5+ star rating
- 6+ months active at Tier 2
- Engagement threshold met
- 3+ vouches from other partners
- Zero policy violations

---

## Network Brands

| Network | Focus | Target |
|---------|-------|--------|
| **Recess** | Movement, fitness, play | Kids & families |
| **TCA** (The Creative Alliance) | Arts, trades, skills, enrichment | All ages |
| **Homeschool** | Educational activities | Homeschool families |

---

## Community Pass / Joy Coins

See COMMUNITY-PASS.md (private repo) for full membership model, pricing, and Joy Coin mechanics.

**Quick reference:**
- Community Pass is a membership subscription ($49/mo base, $39/mo per additional allocation of 12 Joy Coins)
- Joy Coins are non-transferable, expire monthly, have no cash value
- Businesses set their own coin price (1-3 coins per activity)
- Revenue share: 75% business pool / 25% LocalLane (distributed by scan volume)
- Unused coins flow to Scholarship Pool

See STRIPE-CONNECT.md for payment implementation phases.

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

## Repository Structure

```
Spec-Repo/
├── README.md                      # This file
├── ARCHITECTURE.md                # Technical patterns
├── STYLE-GUIDE.md                 # Visual standards
├── DECISIONS.md                   # Decision log
├── WORKFLOW.md                    # Development workflow
├── BUILD-PROTOCOL.md              # Universal build sequence
├── STATUS-TRACKER.md              # Project status and session logs
├── STRIPE-CONNECT.md              # Stripe integration spec
├── RECOMMENDATION-SYSTEM.md       # Nod/Story/Vouch system
├── ENTITY-SYSTEM.md               # Archetype and goals system
├── MYLANE.md                      # MyLane user dashboard spec
├── TIER-SYSTEM.md                 # Business tier definitions
├── USER-TIERS.md                  # User tier definitions
├── ADMIN-ARCHITECTURE.md          # Admin panel structure
├── BUSINESS-DASHBOARD-v2.md       # Business Dashboard strategy spec
├── archetypes/
│   ├── event-node.md              # Event Node assessment (needs repositioning)
│   ├── community-org-engine.md   # Community Organization Engine — churches, nonprofits, mutual aid
│   ├── donut-shop-node.md        # Donut shop archetype (Phase 0 holding doc)
│   ├── fitness-recess-node.md    # Fitness/Recess archetype (Phase 0 holding doc)
│   └── archive/
│       └── _template.md           # Archived — replaced by NODE-PLAYBOOK.md
└── checklists/
    ├── LAUNCH-CHECKLIST.md        # Master launch roadmap (5 phases)
    └── farm-program-launch.md     # Active Phase 1 working checklist
```

---

## Related Repositories

| Repo | Purpose | URL |
|------|---------|-----|
| **community-node** | The main hub app | github.com/withdoron/community-node |
| **events-node** | Partner node for event organizers | github.com/withdoron/events-node |
| **property-pulse** | Family property management app | github.com/withdoron/property-pulse |
| **contractor-daily** | Contractor daily logging app | github.com/withdoron/contractor-daily |
| **Spec-Repo** | This specification repo | github.com/withdoron/Spec-Repo |
| **locallane-private** | Strategy docs (private) | github.com/withdoron/locallane-private |

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

## Contact

- **Owner:** Doron
- **Technical Lead:** Claude + Cursor
- **Email:** doron.bsg@gmail.com

---

Additional strategy and business documents are maintained privately:

| Document | Repo | Purpose |
|----------|------|---------|
| USER-GROUPS-v1.md | locallane-private | User Groups spec (Phase 4) |
| NETWORK-POSTS.md | locallane-private | Network Posts spec — phased communication feature for networks (DEC-051) |
| MYLANE-DYNAMIC-LAYOUT.md | locallane-private | MyLane dynamic section ordering based on user engagement state |

---

*This spec is the external brain for LocalLane development. Always reference before implementing.*

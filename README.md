# LocalLane Ecosystem Specification

> Source of truth for the LocalLane platform architecture, design standards, and development workflows.

---

## Quick Links

| Document | Purpose |
|----------|---------|
| [ARCHITECTURE.md](./platform/ARCHITECTURE.md) | Technical patterns, data models, vitality data layer |
| [STYLE-GUIDE.md](./platform/STYLE-GUIDE.md) | Visual design standards (Gold Standard) |
| [TIER-SYSTEM.md](./platform/TIER-SYSTEM.md) | Tier definitions and feature matrix |
| [RECOMMENDATION-SYSTEM.md](./platform/RECOMMENDATION-SYSTEM.md) | Trust-first Nod/Story/Vouch system |
| [ENTITY-SYSTEM.md](./platform/ENTITY-SYSTEM.md) | Archetype and goals-based dashboard |
| [MYLANE.md](./platform/MYLANE.md) | MyLane spec — user dashboard with organism placement |
| [USER-TIERS.md](./platform/USER-TIERS.md) | User tier definitions (Free/Member) |
| [STATUS-TRACKER.md](./platform/STATUS-TRACKER.md) | Project status, roadmap, and session logs |
| [STRIPE-CONNECT.md](./platform/STRIPE-CONNECT.md) | Stripe integration (Community Pass model) |
| [BUILD-PROTOCOL.md](./platform/BUILD-PROTOCOL.md) | Universal build sequence — how every feature ships. ✅ Current |
| [BUSINESS-DASHBOARD-v2.md](./platform/BUSINESS-DASHBOARD-v2.md) | Business Dashboard v2 strategy spec (replaces INTEGRATION-PLAN.md) |
| [ADMIN-ARCHITECTURE.md](./platform/ADMIN-ARCHITECTURE.md) | Admin panel structure |
| [DECISIONS.md](./platform/DECISIONS.md) | Decision log with context |
| [WORKFLOW.md](./platform/WORKFLOW.md) | Development workflow and session SOP |
| [SPEC-PROTOCOL.md](./platform/SPEC-PROTOCOL.md) | How specs are structured, where they live, how they get written |
| [context/PROJECT-BRAIN.md](./context/PROJECT-BRAIN.md) | Universal AI briefing — project identity, philosophy, working style |
| [context/ACTIVE-CONTEXT.md](./context/ACTIVE-CONTEXT.md) | Current state — overwritten each session |
| [context/SESSION-LOG.md](./context/SESSION-LOG.md) | Running timeline — append-only session history |

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

### The Community + Steward Model

**Community** = A local service area (city/town/county)
- Prototype: Greater Eugene-Springfield
- Scaling unit for expansion

**Steward** = Human curator per Community
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

See [TIER-SYSTEM.md](./platform/TIER-SYSTEM.md) for complete documentation.

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

See `private/platform/COMMUNITY-PASS.md` for full membership model, pricing, and Joy Coin mechanics.

**Quick reference:**
- Community Pass is a membership subscription ($45/mo base per DEC-083, $39/mo per additional allocation of 12 Joy Coins)
- Joy Coins are non-transferable, expire monthly, have no cash value
- Businesses set their own coin price (1-3 coins per activity)
- Revenue share: 75% business pool / 25% LocalLane (distributed by scan volume)
- Unused coins flow to Scholarship Pool

See [STRIPE-CONNECT.md](./platform/STRIPE-CONNECT.md) for payment implementation phases.

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
- See [STYLE-GUIDE.md](./platform/STYLE-GUIDE.md) for complete details

---

## Current Implementation Status

### Event Node (Partner Node Template)
- ✅ Event creation/edit/delete
- ✅ Sync to Community Node
- ✅ Network tagging (Recess, etc.)
- ✅ Joy Coins integration (UI)
- ✅ Dashboard with stats
- ✅ Joy Coins revenue display
- ⏳ Organizer name display (shows "Event Spoke" instead of business name)

### Community Node
- ✅ Browse Events view
- ✅ Event detail modal
- ✅ Quick filters (All, This Weekend, Today)
- ✅ Joy Coins badge display
- ✅ Network badge display
- ⚠️ Admin Panel (light theme - needs dark)
- ⏳ User registration/accounts
- ⏳ Community Pass signup
- ⏳ RSVP functionality
- ⏳ Business directory (deferred to post-pilot)

---

## Repository Structure

Reorganized 2026-04-29 into the platform/cross-space/spaces shape per [SPEC-PROTOCOL.md](./platform/SPEC-PROTOCOL.md) §3.

```
Spec-Repo/
├── README.md                      # This file
├── platform/                      # Whole-platform technical specs
│   ├── SPEC-PROTOCOL.md           # How specs are structured + lifecycle
│   ├── ARCHITECTURE.md            # Technical patterns
│   ├── STYLE-GUIDE.md             # Visual standards (Gold Standard)
│   ├── DECISIONS.md               # Decision log
│   ├── WORKFLOW.md                # Development workflow
│   ├── BUILD-PROTOCOL.md          # Universal build sequence
│   ├── STATUS-TRACKER.md          # Project status and session logs
│   ├── STRIPE-CONNECT.md          # Stripe integration spec
│   ├── RECOMMENDATION-SYSTEM.md   # Nod/Story/Vouch system
│   ├── ENTITY-SYSTEM.md           # Archetype + goals system
│   ├── MYLANE.md                  # MyLane user dashboard spec
│   ├── TIER-SYSTEM.md             # Business tier definitions
│   ├── USER-TIERS.md              # User tier definitions
│   ├── ADMIN-ARCHITECTURE.md      # Admin panel structure
│   ├── BUSINESS-DASHBOARD-v2.md   # Business Dashboard strategy spec
│   ├── ... (other platform-level docs)
│   ├── audits/                    # Whole-platform audits (e.g. drift reports)
│   ├── checklists/                # Cross-cutting launch checklists
│   └── schemas/                   # JSON schemas (e.g. PlatformConfig)
├── cross-space/                   # Specs about how spaces interact (placeholder)
├── spaces/                        # Per-space technical specs
│   ├── field-service/             # Contractor / field-service workspace
│   │   ├── FINANCIAL-WORKFLOW-SPEC.md
│   │   ├── JOBS-MIGRATION-PLAN.md
│   │   ├── farm-program-launch.md
│   │   └── audits/
│   │       ├── CURRENT-STATE-AUDIT.md
│   │       └── TOGGLE-AUDIT.md
│   ├── team/                      # Sports/coaching workspace ("Playmaker" brand)
│   │   ├── PLAYMAKER-ROLES-AND-ONBOARDING-SPEC.md
│   │   └── audits/
│   ├── finance/                   # Personal finance workspace (placeholder)
│   ├── property-management/       # Property management workspace (placeholder)
│   ├── frequency-station/         # Community music space (proposed)
│   └── meal-prep/                 # Meal prep workspace (proposed)
│       └── audits/READINESS-AUDIT.md
├── archetypes/                    # Pre-space node concepts
│   ├── event-node.md
│   ├── community-org-engine.md
│   ├── dog-walker-node.md, donut-shop-node.md, fitness-recess-node.md
│   ├── house-cleaner-node.md, landscaper-node.md, mechanic-node.md
│   └── archive/_template.md
└── context/                       # Session-tracking meta-docs
    ├── PROJECT-BRAIN.md
    ├── ACTIVE-CONTEXT.md
    ├── SESSION-LOG.md
    └── SHIP-IT-PROMPT.md
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
| **private** | Strategy docs (private) | github.com/withdoron/private |

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

See [WORKFLOW.md](./platform/WORKFLOW.md) for details.

---

## Contact

- **Owner:** Doron
- **Technical Lead:** Claude + Cursor
- **Email:** doron.bsg@gmail.com

---

Additional strategy and business documents are maintained privately (in the `withdoron/private` repo). After the 2026-04-29 reorg, paths within private follow the same platform/cross-space/spaces shape:

| Document | Path within private repo | Purpose |
|----------|--------------------------|---------|
| ORGANISM-CONCEPT.md | platform/ | North star vision — the Organism |
| COMMUNITY-PASS.md | platform/ | Membership model, Joy Coins, revenue share |
| LEGAL-RESEARCH.md | platform/ | Oregon regulatory analysis |
| LEGAL-CONSULTATION-PREP.md | platform/ | Lawyer meeting prep |
| BOOK-FRAMEWORKS.md | platform/ | Marketing/business frameworks research |
| NODE-LAB-MODEL.md | platform/ | Node development model (DEC-047) |
| NODE-PLAYBOOK.md | platform/ | Node registry and lifecycle |
| SEEDLING-TRACKER.md | platform/ | Weekly gardener's dashboard |
| DASHBOARD-WORKSPACES.md | platform/ | Workspace engine + Play Trainer implementation spec |
| SILVER-BARTER-CONCEPT.md | cross-space/ | Silver barter directory model (superseded by SILVER-DIME-LAYER) |
| SILVER-DIME-LAYER.md | cross-space/ | Physical value layer concept |
| USER-GROUPS-v1.md | cross-space/ | User Groups spec (Phase 4) |
| FIELD-SERVICE-ENGINE.md | spaces/field-service/ | Multi-vertical field service strategy |
| FINANCIAL-WORKFLOW-INTENT.md | spaces/field-service/ | Strategic context for the financial workflow spec |
| PERSONAL-FINANCE-NODE.md | spaces/finance/ | Financial Node concept |
| BJJ-RANKED-QUEUE.md | spaces/team/ | Competition workspace seed packet |
| BOMB-SQUAD-GROWERS.md | seedlings/ | Dormant cannabis brand strategy |

---

*This spec is the external brain for LocalLane development. Always reference before implementing.*


# Technical Architecture

> Detailed technical design for the LocalLane node ecosystem.

---

## Overview

LocalLane is a **federated system** of interconnected applications ("nodes"). Each node is:

- **Independently deployable** — can be updated without affecting others
- **Loosely coupled** — communicates via defined APIs, not shared code
- **Hierarchical** — child nodes pull config from parents, push data up

---

## Node Communication Patterns

> **Current Status (2026-02-11):** Nodes are standalone Base44 apps with no sync dependencies during the lab phase. The patterns below are preserved as future reference for when mature nodes integrate into LocalLane. See NODE-LAB-MODEL.md (private repo) for the current development model.

**Future integration patterns (not yet implemented):**
- Config Sync — parent pushes config (networks, categories, settings) to child nodes
- Data Push — child nodes push created content (events, listings) to parent
- Health Check — parent monitors child node availability
- Retry with backoff — resilient sync when nodes are temporarily unreachable

These patterns become relevant during Phase 6 (Integration Planning) of the Node Lifecycle defined in NODE-PLAYBOOK.md. Until a node reaches merge maturity, it operates independently.

---

## Data Models

### Core Entities

#### User

#### User

```javascript
{
  id: "user-123",
  email: "user@example.com",
  name: "Jane Doe",
  createdAt: "2026-01-15T10:00:00Z",
  
  // Trust metrics
  trustScore: 4.2,
  ratingsGiven: 15,
  ratingsReceived: 3,
  
  // Relationships
  homeNodeId: "community-portland",  // Primary community
  businessIds: ["biz-456"],          // Businesses they manage
  communityPassId: "pass-789",      // Community Pass membership
  joyCoinsBalance: 0                // Current Joy Coins balance
}
```

#### Business

```javascript
{
  id: "biz-456",
  name: "Portland Jiu-Jitsu",
  archetypeId: "martial-arts",
  tier: 2,
  
  // Location
  address: "123 Main St",
  city: "Portland",
  coordinates: { lat: 45.5, lng: -122.6 },
  
  // Trust
  trustScore: 4.7,
  totalRatings: 89,
  
  // Relationships
  ownerUserId: "user-123",
  communityNodeId: "community-portland",
  partnerNodeId: null,  // Only if Tier 3
  
  // Features
  acceptsJoyCoins: true,
  
  // Config
  networks: ["recess", "tca"]
}
```

#### Event

```javascript
{
  id: "evt-789",
  title: "Family Movie Night",
  description: "...",
  
  // Timing
  date: "2026-02-14T18:00:00Z",
  endDate: "2026-02-14T21:00:00Z",
  timezone: "America/Los_Angeles",
  
  // Location
  location: "Community Center",
  address: "456 Oak Ave",
  isVirtual: false,
  virtualLink: null,
  
  // Relationships
  sourceNodeId: "events-node-001",
  communityEventId: "comm-evt-456",  // ID in community node
  businessId: "biz-456",
  
  // Categorization
  networks: ["recess", "homeschool"],
  categories: ["entertainment", "family"],
  
  // Joy Coins (Community Pass)
  joy_coin_enabled: true,
  joy_coin_cost: 1,  // How many coins to attend
  
  // Attendance
  rsvpCount: 23,
  capacity: 50,
  
  // Status
  status: "published",  // draft, published, cancelled
  createdAt: "2026-01-20T12:00:00Z",
  updatedAt: "2026-01-25T09:00:00Z"
}
```

#### Joy Coins (Community Pass)

```javascript
{
  id: "user-123",
  // Balance tracked per user (JoyCoins entity in Base44)
  current_balance: 7,
  // Monthly allocation from Community Pass subscription; unused coins expire
  // Redemptions recorded in JoyCoinTransactions (PunchPass* entities deleted from Base44 per DEC-028)
}
```

#### Config (Community Node)

```javascript
{
  version: "1.2",
  lastUpdated: "2026-01-27T12:00:00Z",
  
  networks: [
    { id: "recess", name: "Recess", description: "Family fun network", active: true },
    { id: "tca", name: "TCA", description: "TCA community", active: true },
    { id: "homeschool", name: "Homeschool Network", description: "For homeschool families", active: true }
  ],
  
  categories: [
    { id: "sports", name: "Sports & Fitness", icon: "🏃" },
    { id: "education", name: "Education", icon: "📚" },
    { id: "entertainment", name: "Entertainment", icon: "🎬" },
    { id: "food", name: "Food & Dining", icon: "🍕" }
  ],
  
  // Community Pass / Joy Coins: membership subscription; coins per allocation (see STRIPE-CONNECT.md)
  
  tierFeatures: {
    1: ["basic-listing", "post-events"],
    2: ["basic-listing", "post-events", "accept-joy-coins", "analytics"],
    3: ["basic-listing", "post-events", "accept-joy-coins", "analytics", "partner-node", "custom-branding", "priority-search", "trust-badge"]
  }
}
```


## Security Considerations

### Authentication

**Phase 1 (Pilot):**
- Base44 built-in auth for user accounts
- API keys for node-to-node communication (simple, shared secrets)

**Phase 2+:**
- JWT tokens with short expiry
- Refresh token rotation
- OAuth for third-party integrations

### Node-to-Node Auth

Each child node has a secret key registered with its parent:

```javascript
// Child node making request to parent
const response = await fetch(`${PARENT_URL}/api/v1/events`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Node-ID': NODE_ID,
    'X-Node-Secret': NODE_SECRET
  },
  body: JSON.stringify(payload)
});
```

### Data Validation

All incoming data must be validated:

```javascript
// Example: Validate event before publishing
function validateEvent(event) {
  const errors = [];
  
  if (!event.title || event.title.length < 3) {
    errors.push({ field: 'title', message: 'Title must be at least 3 characters' });
  }
  
  if (!event.date || new Date(event.date) < new Date()) {
    errors.push({ field: 'date', message: 'Date must be in the future' });
  }
  
  if (event.networks && !Array.isArray(event.networks)) {
    errors.push({ field: 'networks', message: 'Networks must be an array' });
  }
  
  return errors;
}
```

---

## Observability

### Logging

Every node should log:

- **INFO:** Normal operations (event created, config synced)
- **WARN:** Recoverable issues (config sync failed, using cache)
- **ERROR:** Failures requiring attention (API errors, validation failures)

### Key Metrics (Future)

| Metric | Description |
|--------|-------------|
| `events.created` | Events created per hour |
| `events.synced` | Events successfully pushed to parent |
| `config.sync.success` | Config sync success rate |
| `joycoins.redemptions` | Joy Coins redemptions per day |
| `api.latency` | Response time by endpoint |
| `api.errors` | Error rate by endpoint |

---

## Deployment

### Current Flow

```
Cursor (edit) → GitHub (push) → Base44 (auto-sync) → Manual Publish → Live
```

### Environment Management

| Environment | Purpose | URL Pattern |
|-------------|---------|-------------|
| **Preview** | Testing in Base44 editor | N/A (editor only) |
| **Production** | Live app | `*.base44.app` |

### Future: Staging Environment

When we have real users, we'll add:

```
GitHub main → Base44 Production
GitHub dev → Base44 Staging (separate app)
```

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

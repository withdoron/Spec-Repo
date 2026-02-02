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

### Pattern 1: Config Sync (Parent → Child)

Child nodes need configuration from their parent (networks, categories, settings).

```
┌─────────────────┐                    ┌─────────────────┐
│  COMMUNITY      │                    │  EVENT NODE     │
│  NODE           │                    │                 │
│                 │    GET /config     │                 │
│  Config Store   │◀───────────────────│  On startup     │
│  - networks     │                    │  Every 5 min    │
│  - categories   │────────────────────▶  Cache locally  │
│  - settings     │    JSON response   │                 │
└─────────────────┘                    └─────────────────┘
```

**Implementation:**

```javascript
// Event Node: Fetch config on startup and periodically
const CONFIG_REFRESH_INTERVAL = 5 * 60 * 1000; // 5 minutes

async function syncConfig() {
  try {
    const response = await fetch(`${COMMUNITY_NODE_URL}/api/v1/config`);
    const { version, config } = await response.json();
    
    // Only update if version changed
    if (version !== localConfig.version) {
      localStorage.setItem('config', JSON.stringify({ version, config }));
      applyConfig(config);
    }
  } catch (error) {
    // Use cached config if parent unreachable
    console.warn('Config sync failed, using cache');
  }
}

// Run on startup and every 5 minutes
syncConfig();
setInterval(syncConfig, CONFIG_REFRESH_INTERVAL);
```

### Pattern 2: Data Push (Child → Parent)

When a child node creates content (event, listing), it pushes to parent.

```
┌─────────────────┐                    ┌─────────────────┐
│  EVENT NODE     │                    │  COMMUNITY      │
│                 │                    │  NODE           │
│  User creates   │   POST /events     │                 │
│  "Movie Night"  │───────────────────▶│  Aggregated     │
│                 │                    │  Event List     │
│                 │   201 Created      │                 │
│                 │◀───────────────────│  Shows in       │
│                 │   communityId: X   │  calendar       │
└─────────────────┘                    └─────────────────┘
```

**Implementation:**

```javascript
// Event Node: Push event to Community Node
async function publishEvent(event) {
  const payload = {
    sourceNodeId: NODE_ID,
    event: {
      id: event.id,
      title: event.title,
      date: event.date,
      location: event.location,
      description: event.description,
      networks: event.networks,
      acceptsPunchPass: event.acceptsPunchPass,
      punchPassTypes: event.punchPassTypes
    }
  };
  
  const response = await fetch(`${COMMUNITY_NODE_URL}/api/v1/events`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });
  
  const { communityEventId } = await response.json();
  
  // Store the community ID for future updates/deletes
  await updateLocalEvent(event.id, { communityEventId });
}
```

### Pattern 3: Cross-Node Transactions (Punch Pass)

Punch pass verification requires real-time communication.

```
┌──────────┐      ┌─────────────────┐      ┌─────────────────┐
│  USER    │      │  PARTNER NODE   │      │  COMMUNITY      │
│          │      │  (Jiu-Jitsu)    │      │  NODE           │
│          │      │                 │      │                 │
│  Shows   │─────▶│  Scan QR        │      │                 │
│  QR code │      │  Extract passId │      │                 │
│          │      │                 │      │                 │
│          │      │  POST /verify   │─────▶│  Check pass     │
│          │      │  passId, bizId  │      │  valid?         │
│          │      │                 │      │  has punches?   │
│          │      │                 │◀─────│  Deduct punch   │
│          │      │  Show success   │      │  Return result  │
│  ✓       │◀─────│  "7 remaining"  │      │                 │
└──────────┘      └─────────────────┘      └─────────────────┘
```

---

## Data Models

### Core Entities

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
  punchPassIds: ["pass-789"]         // Passes they own
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
  acceptsPunchPass: true,
  acceptedPassTypes: ["recess-10", "tca-5"],
  
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
  
  // Punch Pass
  acceptsPunchPass: true,
  punchPassTypes: ["recess-10"],
  punchCost: 1,  // How many punches to attend
  
  // Attendance
  rsvpCount: 23,
  capacity: 50,
  
  // Status
  status: "published",  // draft, published, cancelled
  createdAt: "2026-01-20T12:00:00Z",
  updatedAt: "2026-01-25T09:00:00Z"
}
```

#### Punch Pass

```javascript
{
  id: "pass-789",
  typeId: "recess-10",
  typeName: "Recess 10-Pack",
  
  // Ownership
  userId: "user-123",
  
  // Balance
  totalPunches: 10,
  remainingPunches: 7,
  
  // Value (for tracking, not real money in Phase 1)
  purchasePrice: 50.00,
  
  // Status
  status: "active",  // active, depleted, expired, refunded
  purchasedAt: "2026-01-10T14:00:00Z",
  expiresAt: "2027-01-10T14:00:00Z",
  
  // History
  transactions: [
    { date: "2026-01-15", businessId: "biz-456", punches: 1, type: "redemption" },
    { date: "2026-01-20", businessId: "biz-789", punches: 2, type: "redemption" }
  ]
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
  
  punchPassTypes: [
    { id: "recess-10", network: "recess", name: "Recess 10-Pack", price: 50, punches: 10, expiryDays: 365 },
    { id: "recess-20", network: "recess", name: "Recess 20-Pack", price: 90, punches: 20, expiryDays: 365 },
    { id: "tca-5", network: "tca", name: "TCA 5-Pack", price: 25, punches: 5, expiryDays: 180 }
  ],
  
  tierFeatures: {
    1: ["basic-listing", "post-events"],
    2: ["basic-listing", "post-events", "accept-punch-pass", "analytics"],
    3: ["basic-listing", "post-events", "accept-punch-pass", "analytics", "partner-node", "custom-branding", "priority-search", "trust-badge"]
  }
}
```

---

## API Reference

### Community Node APIs

#### GET /api/v1/config

Returns current configuration for child nodes.

**Response:**
```json
{
  "version": "1.2",
  "lastUpdated": "2026-01-27T12:00:00Z",
  "config": { /* see Config model above */ }
}
```

#### POST /api/v1/events

Create or update an event from a child node.

**Request:**
```json
{
  "sourceNodeId": "events-node-001",
  "event": { /* see Event model */ }
}
```

**Response:**
```json
{
  "success": true,
  "communityEventId": "comm-evt-456",
  "action": "created"  // or "updated"
}
```

#### DELETE /api/v1/events/:sourceNodeId/:eventId

Remove an event.

**Response:**
```json
{
  "success": true,
  "action": "deleted"
}
```

#### POST /api/v1/punch-pass/verify

Verify and redeem a punch pass.

**Request:**
```json
{
  "passId": "pass-789",
  "businessId": "biz-456",
  "punchesRequested": 1
}
```

**Response (success):**
```json
{
  "valid": true,
  "passTypeName": "Recess 10-Pack",
  "remainingPunches": 6,
  "transactionId": "txn-abc-123"
}
```

**Response (failure):**
```json
{
  "valid": false,
  "reason": "insufficient_punches",  // or "expired", "not_found", "wrong_network"
  "message": "Pass only has 0 punches remaining"
}
```

#### GET /api/v1/businesses

List businesses in the community.

**Query params:**
- `category` — Filter by category ID
- `network` — Filter by network ID
- `tier` — Filter by tier (1, 2, 3)
- `search` — Text search
- `page`, `limit` — Pagination

**Response:**
```json
{
  "businesses": [ /* Business objects */ ],
  "total": 45,
  "page": 1,
  "limit": 20
}
```

#### GET /api/v1/events

List aggregated events from all child nodes.

**Query params:**
- `from`, `to` — Date range
- `network` — Filter by network
- `category` — Filter by category

---

## Error Handling

### Standard Error Response

All APIs return errors in this format:

```json
{
  "error": true,
  "code": "VALIDATION_ERROR",
  "message": "Event title is required",
  "details": {
    "field": "title",
    "constraint": "required"
  }
}
```

### Error Codes

| Code | HTTP Status | Meaning |
|------|-------------|---------|
| `VALIDATION_ERROR` | 400 | Request data invalid |
| `UNAUTHORIZED` | 401 | Missing or invalid auth |
| `FORBIDDEN` | 403 | Not allowed to perform action |
| `NOT_FOUND` | 404 | Resource doesn't exist |
| `CONFLICT` | 409 | Resource already exists |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

### Retry Strategy

Child nodes should implement exponential backoff for failed requests:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response;
      
      // Don't retry client errors (4xx)
      if (response.status >= 400 && response.status < 500) {
        throw new Error(`Client error: ${response.status}`);
      }
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      
      // Exponential backoff: 1s, 2s, 4s
      await sleep(Math.pow(2, attempt) * 1000);
    }
  }
}
```

---

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
| `punchpass.redemptions` | Punch pass redemptions per day |
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

LocalLane's north star is making community health visible (DEC-026). This requires a data pipeline that transforms platform activity into vitality signals.

### Data Signals

The platform already generates the raw signals needed:

| Signal | Source Entity | Already Queried? |
|--------|--------------|-----------------|
| RSVPs created | RSVP | ✅ Yes (MyLane) |
| Event attendance (check-ins) | PunchPassUsage | ✅ Yes (CheckIn) |
| Recommendations given | Recommendation | ✅ Yes (MyLane) |
| Community Pass activity | PunchPass | ✅ Yes (MyLane) |
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

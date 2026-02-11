# Entity System Architecture

> How businesses and organizations onboard, select goals, and unlock dashboard features.

---

## Overview

The Community Node serves multiple types of entities (businesses, organizations, individuals). Each entity:
1. Completes an **Onboarding Wizard**
2. Selects an **Archetype** (what they are)
3. Selects **Goals** (what they want to do)
4. Provides **Details** (based on goals)
5. Chooses a **Plan** (Tier 1, 2, or 3)

The combination of Archetype + Goals + Tier determines what **Dashboard Modules** are visible.

---

## Archetypes

Archetypes define what type of entity is joining the platform.

| Archetype | Icon | Description | Examples |
|-----------|------|-------------|----------|
| **Location/Venue** | 🏪 | Physical space for customers | Gyms, studios, restaurants, venues |
| **Event Organizer** | 🎫 | Hosts events without permanent venue | Festival organizers, meetup hosts |
| **Service Provider** | 💼 | Mobile or professional services | Consultants, contractors, tutors |
| **Community/Non-Profit** | ❤️ | Groups, churches, causes | Churches, clubs, charities, associations |
| **Product Seller** | 🏪 | Physical or digital products | Farmers, artisans, creators |

### Archetype Configuration (Admin)

Archetypes are configured in the Admin Dashboard, not hardcoded. Each archetype has:
- Name
- Icon
- Description
- Available Goals (which goals make sense for this archetype)
- Required Fields (what details are needed)

---

## Goals

Goals determine what features an entity wants to use. Goals unlock Dashboard Modules.

| Goal | Description | Unlocks |
|------|-------------|---------|
| **Host Events** | Create and manage events | Event Module |
| **Accept Joy Coins** | Accept Joy Coins from Community Pass members | Joy Coins Module |
| **List in Directory** | Appear in business directory | Directory Profile Module |
| **Manage Bookings** | Accept appointments/reservations | Booking Module (future) |
| **Sell Products** | List products for sale | Marketplace Module (future) |
| **Post Announcements** | Send updates to followers | Announcements Module (future) |

### Goal → Archetype Matrix

Not all goals are relevant for all archetypes:

| Goal | Venue | Event Org | Service | Non-Profit | Seller |
|------|-------|-----------|---------|------------|--------|
| Host Events | ✅ | ✅ | ✅ | ✅ | ⚪ |
| Accept Punch Pass | ✅ | ✅ | ⚪ | ⚪ | ⚪ |
| List in Directory | ✅ | ⚪ | ✅ | ✅ | ✅ |
| Manage Bookings | ✅ | ⚪ | ✅ | ⚪ | ⚪ |
| Sell Products | ⚪ | ⚪ | ⚪ | ⚪ | ✅ |
| Post Announcements | ✅ | ✅ | ⚪ | ✅ | ⚪ |

✅ = Available  ⚪ = Not typically relevant

---

## Onboarding Wizard Flow

```
Step 1: ARCHETYPE
"How do you serve the community?"
→ User selects one archetype
→ This filters available goals in Step 2

Step 2: GOALS (moved before Details)
"What do you want to do on Local Lane?"
→ User selects one or more goals
→ This determines what details are needed

Step 3: DETAILS
"Tell us about your organization"
→ Fields shown based on archetype + goals
→ Examples:
  - If "Host Events": Event-related fields
  - If "List in Directory": Address, hours, photos
  - If "Accept Joy Coins": Payment setup (future)

Step 4: PLAN
"Choose your plan"
→ Show tiers based on goals selected
→ Some goals require Tier 2+ (e.g., Accept Punch Pass)

Step 5: REVIEW
"Review and confirm"
→ Summary of selections
→ Submit to create account
```

---

## Tier System

Tiers determine feature limits and capabilities within each module.

| Tier | Code Value | Display Name | Cost | Key Differences |
|------|------------|--------------|------|-----------------|
| 1 | `basic` | Basic | Free | Basic features, events go to review queue |
| 2 | `standard` | Standard | $79/mo | Full features, auto-publish, Joy Coins acceptance |
| 3 | `partner` | Partner | $Y/mo + Earned | Own standalone app (Partner Node) |

**Note:** Always use the code values (`basic`, `standard`, `partner`) in database and code.

### Tier → Feature Matrix

| Feature | Basic (Tier 1) | Standard (Tier 2) | Partner (Tier 3) |
|---------|----------------|-------------------|------------------|
| Create Events | ✅ (reviewed) | ✅ (auto-publish) | ✅ (own node) |
| Event Analytics | ❌ | ✅ | ✅ |
| Accept Punch Pass | ❌ | ✅ | ✅ |
| Punch Pass Earnings | ❌ | ✅ | ✅ |
| Check-in Feature | ❌ | ✅ | ✅ |
| Directory Listing | Basic | Enhanced | Enhanced + Badge |
| Own Branding | ❌ | ❌ | ✅ |
| Own Domain | ❌ | ❌ | ✅ |
| API Access | ❌ | ❌ | ✅ |

### Goal → Tier Requirements

Some goals require minimum tier:

| Goal | Minimum Tier | Code Value |
|------|--------------|------------|
| Host Events | Tier 1 | `basic` |
| List in Directory | Tier 1 | `basic` |
| Accept Joy Coins | Tier 2 | `standard` |
| Manage Bookings | Tier 2 | `standard` |
| Sell Products | Tier 2 | `standard` |
| Post Announcements | Tier 1 | `basic` |

---

## Dashboard Modules

Each module is a section of the Entity Dashboard. Modules are shown/hidden based on Goals selected.

### Event Module

**Unlocked by:** "Host Events" goal

**Tier 1 Features:**
- Create Event (goes to review queue)
- View My Events
- Basic RSVP list

**Tier 2 Features (adds):**
- Auto-publish events
- Analytics dashboard
- Check-in feature
- Recurring events
- Waitlist management

**Components:**
- Event list table
- Create/Edit event form
- RSVP management
- Analytics charts (Tier 2)

### Joy Coins Module

**Unlocked by:** "Accept Joy Coins" goal
**Minimum Tier:** 2 (Standard)

**Features:**
- Joy Coins earnings dashboard
- Check-in scan history
- Revenue share payout information
- Community Pass network settings

**Components:**
- Earnings summary cards
- Scan history table
- Revenue share overview

### Directory Profile Module

**Unlocked by:** "List in Directory" goal

**Tier 1 Features:**
- Basic profile (name, description)
- Contact info
- Single photo

**Tier 2 Features (adds):**
- Photo gallery
- Hours of operation
- "We're Hiring" toggle
- Enhanced SEO

**Components:**
- Profile editor
- Photo uploader
- Hours editor

---

## Data Model

### Entity (Business/Organization)

```javascript
Entity {
  id: UUID,
  userId: UUID,              // Owner's user account
  
  // Onboarding selections
  archetypeId: string,       // 'venue', 'event_organizer', etc.
  goals: string[],          // ['host_events', 'accept_joy_coins']
  tier: number,             // 1, 2, or 3
  
  // Basic info
  name: string,
  description: string,
  email: string,
  phone: string,
  website: string,
  
  // Location (if applicable)
  address: Address,
  coordinates: { lat, lng },
  
  // Directory info
  hours: Hours[],
  photos: string[],
  categories: string[],
  networks: string[],       // ['recess', 'homeschool']
  
  // Status
  status: 'pending' | 'active' | 'suspended',
  trustedStatus: boolean,    // Can auto-publish
  
  // Punch Pass
  acceptsPunchPass: boolean,
  punchPassTypes: string[],  // Which pass types accepted
  
  // Partner Node (Tier 3)
  partnerNodeId: UUID | null,
  
  // Timestamps
  createdAt: Date,
  updatedAt: Date,
}
```

### Archetype Configuration

```javascript
ArchetypeConfig {
  id: string,                // 'venue', 'event_organizer', etc.
  name: string,              // 'Location / Venue'
  icon: string,              // Icon identifier
  description: string,
  availableGoals: string[],  // Which goals can be selected
  requiredFields: string[],  // What details are required
  sortOrder: number,
}
```

### Goal Configuration

```javascript
GoalConfig {
  id: string,                // 'host_events'
  name: string,              // 'Host Events'
  description: string,
  minimumTier: number,       // 1, 2, or 3
  unlocksModules: string[],  // ['event_module']
  requiredFields: string[],  // Additional fields needed
}
```

---

## Admin Dashboard

The Admin Dashboard controls platform-wide settings.

### Tabs

| Tab | Purpose |
|-----|---------|
| **Businesses** | Manage all entities, tiers, status |
| **Locations** | Manage geographic areas |
| **Settings** | Platform configuration |
| **Spokes** | Manage Partner Nodes (Tier 3) |
| **Categories** | Configure event/business categories |
| **Networks** | Configure community networks (Recess, TCA, etc.) |

### Categories Configuration

Categories are NOT hardcoded. Admin configures:

```javascript
Category {
  id: UUID,
  name: string,              // 'Sports & Active'
  type: 'event' | 'business' | 'both',
  icon: string,
  sortOrder: number,
  active: boolean,
}
```

### Networks Configuration

```javascript
Network {
  id: UUID,
  name: string,              // 'Homeschool Community'
  slug: string,              // 'homeschool'
  description: string,
  icon: string,
  active: boolean,
}
```

---

## Partner Node (Tier 3) Integration

When an entity upgrades to Tier 3:

1. **Partner Node Created** — Standalone app instance
2. **Config Sync** — Node pulls categories, networks, settings from Community Node
3. **Data Sync** — Events created in Partner Node sync UP to Community Node
4. **Branding** — Partner Node can have custom branding, domain

### Sync Rules

| Data | Direction | Frequency |
|------|-----------|-----------|
| Categories | Community → Partner | Every 5 min |
| Networks | Community → Partner | Every 5 min |
| Settings | Community → Partner | Every 5 min |
| Events | Partner → Community | On create/update |
| Entity Profile | Partner → Community | On update |

---

## Implementation Priority

For pilot, focus on:

1. ✅ Onboarding Wizard (already built)
2. ⏳ Event Module (copy from Event Node)
3. ⏳ Goal-based module visibility
4. ⏳ Tier-based feature gating
5. ⏸️ Directory Profile Module (post-pilot)
6. ⏸️ Other modules (future)

---

*This document defines how entities join the platform and what features they unlock.*
